---
title: "Terraform: Alternative to the Template provider on Apple M1 MBP"
date: 2022-08-05T17:52:07+01:00
draft: false
tags: ["terraform"]
---
We ran into an issue while applying our Terraform infrastructure on a M1 Mac where we were making use of the [Terraform Provider Template](https://github.com/hashicorp/terraform-provider-template).

When applying it, we were getting the following error:
```
template v2.2.0 does not have a package available for your current platform, darwin_arm64
``` 

Since the provider is archived, we need to find an alternative.

_What does archiving mean?_
Per Terraform [Archiving Providers](https://www.terraform.io/internals/archiving) documentation.

> - The code repository and all commit, issue, and PR history will still be available.
> - Existing released binaries will remain available on the releases site.
> - Documentation for the provider will remain on the Terraform website.
> - Issues and pull requests are not being monitored, merged, or added.
> - No new releases will be published.
> - Nightly acceptance tests may not be run.

_So what alternatives do we have instead of the deprecated provider?_ 

Let's look at an example.

## Resource using the deprecated Template provider

Let's say we have the following resource - a grafana dashboard json that we store in our Terraform code.

```hcl
data "template_file" "grafana_json" {
  template = file("${path.module}/grafana_dashboard.json")
  vars = {
    title                      = var.monitoring_title
    monitoring_datasource_name = var.monitoring_datasource_name
  }
}
```

And the grafana dashboard Terraform resource:

```hcl
resource "grafana_dashboard" "metrics" {
  config_json = data.template_file.grafana_json.rendered
  folder      = var.monitoring_folder
}
```

When you try to apply this piece of code it will throw the aforementioned error.

## Updating to the built-in `templatefile` Terraform function

We can make use of the built-in `templatefile` [Terraform function](https://www.terraform.io/language/functions/templatefile) that:

> _`templatefile` reads the file at the given path and renders its content as a template using a supplied set of template variables._

The function uses the format:
`templatefile(path, vars)`

In our case the `path` is the path to the grafana json file.

And `vars` contains all the variables that we need to use for the grafana dashboard json file.

> _The "vars" argument must be a map. Within the template file, each of the keys in the map is available as a variable for interpolation. The template may also use any other function available in the Terraform language, except that recursive calls to templatefile are not permitted. Variable names must each start with a letter, followed by zero or more letters, digits, or underscores._

And our new code looks like this:
```hcl
resource "grafana_dashboard" "metrics" {
  config_json = templatefile("${path.module}/grafana_dashboard.json", {
    title                      = var.monitoring_title
    monitoring_datasource_name = var.monitoring_datasource_name
  })
  folder = var.monitoring_folder
}
```

Thanks to the new `templatefile` function, we can get rid of the `template_file` data source.

This means at this point we no longer rely on the hashicorp/template provider and we can apply our infrastructure changes.

At this point you can apply the infrastructural changes, but it still might not work and throw the error and this is because if the infrastructure has already been initialized and applied previously we have a record of the deprecated provider stored in the lock file.

## The template provider is still in the `.terraform.lock.hcl` file

If you run `terraform init` and still see that the template provider is being installed:

```console
Initializing modules...

Initializing the backend...

Initializing provider plugins...
- Reusing previous version of grafana/grafana from the dependency lock file
...
- Reusing previous version of hashicorp/template from the dependency lock file
...
- Using previously-installed hashicorp/template v2.2.0

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

Then check the Dependency Lock File, `terraform.hcl.lock` file. If you still see it there:

```hcl
provider "registry.terraform.io/hashicorp/template" {
  version = "2.2.0"
  hashes = [
    "h1:0wlehNaxBX7GJQnPfQwTNvvAf38Jm0Nv7ssKGMaG6Og=",
    "zh:01702196f0a0492ec07917db7aaa595843d8f171dc195f4c988d2ffca2a06386",
    "zh:09aae3da826ba3d7df69efeb25d146a1de0d03e951d35019a0f80e4f58c89b53",
    "zh:09ba83c0625b6fe0a954da6fbd0c355ac0b7f07f86c91a2a97849140fea49603",
    "zh:0e3a6c8e16f17f19010accd0844187d524580d9fdb0731f675ffcf4afba03d16",
    "zh:45f2c594b6f2f34ea663704cc72048b212fe7d16fb4cfd959365fa997228a776",
    "zh:77ea3e5a0446784d77114b5e851c970a3dde1e08fa6de38210b8385d7605d451",
    "zh:8a154388f3708e3df5a69122a23bdfaf760a523788a5081976b3d5616f7d30ae",
    "zh:992843002f2db5a11e626b3fc23dc0c87ad3729b3b3cff08e32ffb3df97edbde",
    "zh:ad906f4cebd3ec5e43d5cd6dc8f4c5c9cc3b33d2243c89c5fc18f97f7277b51d",
    "zh:c979425ddb256511137ecd093e23283234da0154b7fa8b21c2687182d9aea8b2",
  ]
}
```

Check who is requiring the provider (maybe it's still being used in the code elsewhere). This can be done by running the `terraform providers` command, which:

> The terraform providers command shows information about the provider requirements of the configuration in the current working directory, as an aid to understanding where each requirement was detected from.

```console
Providers required by configuration:
.
├── provider[registry.terraform.io/grafana/grafana]
├── ...
├── ...
├── ...
├── module.grafana
│   └── provider[registry.terraform.io/grafana/grafana]
└── module.module
    ├── ...

Providers required by state:

    provider[registry.terraform.io/hashicorp/template]

    provider[registry.terraform.io/grafana/grafana]

```
In this case we can see that the template provider is required by the state.

In order to get rid of this dependency, make sure you update Terraform to any versions greater than **v.1.1.3.** and this is because they fixed the following issue: https://github.com/hashicorp/terraform/pull/30192 in version 1.1.3.

In order to update the lock file and remove the entry for the deprecated template provider, we run `terraform init`.

This is because Terraform relies on two sources for determining the truth: the _configuration_ itself and the _state_. If you remove the dependency on a particular provider from _both_ your configuration and the state then running `terraform init` will remove any existing lock file entry for that provider.

And let's look at the output:

```console
Initializing modules...

Initializing the backend...

Successfully configured the backend "azurerm"! Terraform will automatically
use this backend unless the backend configuration changes.

Initializing provider plugins...

- Reusing previous version of grafana/grafana from the dependency lock file

- Using previously-installed grafana/grafana v1.17.0

Terraform has made some changes to the provider dependency selections recorded
in the .terraform.lock.hcl file. Review those changes and commit them to your
version control system if they represent changes you intended to make.

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

And we see the template provider is no longer there. 

Now you can safely commit the freshly updated [Dependency Lock File](https://www.terraform.io/language/files/dependency-lock). 
