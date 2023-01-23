---
title: "Terraform: Handling the deletion of a non-empty AWS S3 Bucket"
date: 2022-05-25T12:39:03+01:00
draft: false
tags: ["terraform", "aws", "s3", "iac"]
---
This article applies to **Terraform v1.1.4**

When using Terraform to manage your infrastructure you will end up in the situation when you want to remove some resources.

You can do this in several ways, but most of the time you can also just remove the Terraform configuration by commenting it out the code, or removing the calling of the module, run `terraform apply` and it will get rid of the resources.

**Infrastructure Setup**

Assuming an infra with the following setup:

```
- envs
|__ staging
|_____ main.tf
...
|__ prod
|_____ main.tf
...
- modules
|__ awsresources
|_____ s3.tf
...
```

Where `main.tf` file calls the module responsible for creating the AWS resources we need:

```hcl
module "aws" {
  source       = "../../modules/awsresources"
  bucket_name  = "my-bucket"
  fqdn         = "my-bucket"
  deployer_arn = "***"

  tags = {
  ...
  }
}
```

And a simple S3 bucket configured as follows:

```hcl
resource "aws_s3_bucket" "name" {
  bucket = var.bucket_name
  acl    = "private"
  policy = data.aws_team_policy_document.bucket_policy.json

  website {
  }

  force_destroy = false

  tags = var.tags
}
```

We want to destroy these resources, specifically the S3 bucket itself.

**Removal of resources**

If you run `terraform plan` it will mark it nicely as to be destroyed:
```console
# module.module_name.aws_s3_bucket.name will be destroyed
  # (because aws_s3_bucket.static_site is not in configuration)
  - resource "aws_s3_bucket" "static_site" {
      - acl                         = "private" -> null
      - arn                         = "***" -> null
      - bucket                      = "***" -> null
      - bucket_domain_name          = "***" -> null
      - bucket_regional_domain_name = "***" -> null
      - force_destroy               = false -> null
      - hosted_zone_id              = "***" -> null
      - id                          = "***" -> null
      - policy                      = jsonencode(
...
```

But running the `terraform apply` on the same plan and on a non-empty S3 bucket will result in the following ERROR:

```console
╷
│ Error: error deleting S3 Bucket (***): BucketNotEmpty: The bucket you tried to delete is not empty
│ 	status code: 409, request id: ***, host id: ***
│
```

**Let's check the S3 bucket configuration again**

Checking the configuration, we see that we set the flag  `force_destroy = false` in our case. This is actually a very good check in place to have because it protects you from accidental data loss.

From the Terraform aws provider [documentation](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/s3_bucket#force_destroy):

> force_destroy - (Optional, Default:false) A boolean that indicates all objects (including any locked objects) should be deleted from the bucket so that the bucket can be destroyed without error. These objects are not recoverable.

In our case we have to set `force_destroy = true` to allow the bucket to be deleted.

**Pay attention: You must apply this change so state is updated first, before running the destroy command.**

So let's change the value to `true` and apply it on our resource:
```console
 # module.suspended_workspace.aws_s3_bucket.static_site will be updated in-place
  ~ resource "aws_s3_bucket" "name" {
      ~ force_destroy               = false -> true
        id                          = "***"
        tags                        = {
           ...
        }
        # (12 unchanged attributes hidden)
        # (2 unchanged blocks hidden)
    }
```

```console
Apply complete! Resources: 0 added, 1 changed, 0 destroyed.
```

After the setting has been applied successfully, we can start deleting the resources by:
- removing the module from being called, or 
- commenting out the resource 

and run a `terraform apply` again to confirm the destruction of the resources.

Alternatively, you can also run:
`terraform plan -destroy -target=aws_s3_bucket.name`

This time the S3 bucket is deleted successfully.

**What to do if you noticed the error after some resources have already been destroyed?**

So let's say you already started to apply the destruction of the resources and some are successfully destroyed and some are not, including our S3 bucket. What can you do at this point?

_Option 1_ 
You can:
Use the `-target=resource` like below to target the S3 bucket changes only and work only with that resource:

```hcl
terraform plan -target=module.mymodule.aws_s3_bucket.name
terraform apply -target=module.mymodule.aws_s3_bucket.name
```
or

```hcl
terraform plan -target=aws_s3_bucket.name
terraform apply -target=aws_s3_bucket.name
```
As a note, you can add multiple resources in any of the commands if you have multiple S3 buckets that need to be deleted.

OR

_Option 2_ 
You can:
1. Re-apply the configuration essentially re-creating the missing resources
2. Setting the `force_destroy` flag
3. Run `terraform apply` again to destroy the resources

This of course depends on the level of complexity of your infrastructure, in some cases rendering it difficult to do.

_And there you go. Hope you find it useful!_
