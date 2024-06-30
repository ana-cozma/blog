---
title: "Terraform vs Helm for Managing K8s Objects"
date: 2022-08-10T19:34:20+01:00
draft: false
tags: ["kubernetes", "terraform", "helm"]
---
When I started migrating to Kubernetes (K8s) I discovered that I can use Terraform for managing not only the infrastructure, but also I could define the K8s objects in it, but I also could use Helm to handle that. But what would be a good way to handle this?

In this post we will cover Terraform and Helm for managing Kubernetes clusters with some code snippets and an idea on how you can use them together to get you started.

### Structure of the post:
1. [What is Terraform?](#What-is-Terraform?)
2. [Manage Kubernetes Resources via Terraform](#Manage-Kubernetes-Resources-via-Terraform)
3. [What is Helm?](#What-is-Helm?)
4. [Manage Kubernetes Resources via Helm](#Manage-Kubernetes-Resources-via-Helm)
5. [Using Helm and Terraform Together](#Using-Helm-and-Terraform-Together)

## What is Terraform?
[HashiCorp Terraform](https://www.terraform.io/) is an infrastructure as code tool that lets you define both cloud and on-prem resources in human-readable configuration files that you can version, reuse, and share. 

It can manage low-level components like compute, storage, and networking resources, as well as high-level components like DNS entries and SaaS features.

Terraform treats Infrastructure as Code (IaC) meaning teams manage infrastructure setup with configuration files instead of using graphical user interface (think of Azure Portal and the like). 

_So why would you use Terraform?_

Some of the _benefits_ include:

- It allows teams to build, change, and manage the infrastructure in a **safe, consistent, and repeatable way** by defining resource configurations that can be versioned, reused, and shared.

- It supports all major cloud **providers**: Azure, AWS, GCP and many other which you can find [by browsing their registry](https://registry.terraform.io/browse/providers).

> _Providers define individual units of infrastructure, for example compute instances or private networks, as resources. You can compose resources from different providers into reusable Terraform configurations called modules, and manage them with a consistent language and workflow._

You define your providers in the terraform code as follo3ws:
```hcl
terraform {
  required_providers {
    helm = {
      version = "2.5.1"
    }
    azuread = {
      source  = "hashicorp/azuread"
      version = "2.23.0"
    }
    azurerm = {
      source = "hashicorp/azurerm"
    }
  }
```

- It's configuration language is **declarative**: 

> _Meaning that it describes the desired **end-state for your infrastructure**, in contrast to procedural programming languages that require step-by-step instructions to perform tasks. Terraform providers automatically calculate dependencies between resources to create or destroy them in the correct order._

This means that any new team member joining will be able to understand the infrastructure setup you have just by going through the configuration files.

- It's **state** allows you to track resource changes throughout your deployments.

- All configurations are subject to **version control** to safely collaborate on infrastructure.

You can read more on the advantages it brings by looking over the [official use cases](https://www.terraform.io/use-cases/infrastructure-as-code) from the Terraform documentation.

_Terraform - How does it work?_

Terraform follows a simple _workflow_ for managing your infrastructure. So once a new resource or changes to resources are desired, the team will:

**Initialize** the backend by running the `terraform init` command, which will install the plugins Terraform needs to manage the infrastructure.

Output will look something like:
```console
Initializing modules...
(...)
- module1 in ../../modules/module1
- module2 in ../../modules/module2

Initializing the backend...

Successfully configured the backend "azurerm"! Terraform will automatically
use this backend unless the backend configuration changes.

Initializing provider plugins...
- Reusing previous version of hashicorp/kubernetes from the dependency lock file
- Reusing previous version of hashicorp/azuread from the dependency lock file
- Reusing previous version of hashicorp/azurerm from the dependency lock file
- Reusing previous version of hashicorp/helm from the dependency lock file
- Installing hashicorp/helm v2.5.1...
- Installed hashicorp/helm v2.5.1 (signed by HashiCorp)
- Installing hashicorp/kubernetes v2.10.0...
- Installed hashicorp/kubernetes v2.10.0 (signed by HashiCorp)
- Installing hashicorp/azuread v2.23.0...
- Installed hashicorp/azuread v2.23.0 (signed by HashiCorp)
- Installing hashicorp/azurerm v3.10.0...
- Installed hashicorp/azurerm v3.10.0 (signed by HashiCorp)

Partner and community providers are signed by their developers.
If you'd like to know more about provider signing, you can read about it here:
https://www.terraform.io/docs/cli/plugins/signing.html

Terraform has been successfully initialized!

You may now begin working with Terraform. Try running "terraform plan" to see
any changes that are required for your infrastructure. All Terraform commands
should now work.

If you ever set or change modules or backend configuration for Terraform,
rerun this command to reinitialize your working directory. If you forget, other
commands will detect it and remind you to do so if necessary.
```

In the example above we initialize the backend, in our case we store the state file in a container in azure, the providers (azurerm, azuread, helm, kubernetes) and we get a successful message of completion.

**Plan** the changes to be made by running the `terraform plan` command, which will give a review of the 'planned' changes Terraform will make to match your configuration.

During the plan, Terraform will mark which resources will be:
- added with a '+' sign, 
- updated with a '~' sign or 
- deleted with a '-' sign.

In this example Terraform is creating a brand new resource as you can see all attributes are marked with the + sign:

```console
  # module.monitoring.helm_release.prometheus_agent[0] will be created
  + resource "helm_release" "prometheus_agent" {
      + atomic                     = false
      + chart                      = "../../charts/prometheus-agent"
      + cleanup_on_fail            = false
      + create_namespace           = false
      + dependency_update          = false
      + disable_crd_hooks          = false
      + disable_openapi_validation = false
      + disable_webhooks           = false
      + force_update               = false
(...)
```
But in this one, the resource was already previously created, and Terraform is just updating something to it which is marked with the ~ sign:

```console
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  + create
  ~ update in-place

Terraform will perform the following actions:
  # module.grafana.grafana_organization.org will be updated in-place
  ~ resource "grafana_organization" "org" {
      ~ admins       = [
          + "new_admin@grafana.admin",
        ]
        id           = "1"
        name         = "MyOrg"
        # (5 unchanged attributes hidden)
    }
```

Terraform keeps track of your real infrastructure in a _state_ file, which acts as a source of truth for your environment. Meaning it uses this file to determine the changes to make to your infrastructure so that it will match your configuration.

This is also helpful to detect infrastructure drifts between the desired state and the current one.

**Apply** the desired changes by running the `terraform apply` command.

A view from the Terraform official documentation:

![Image description](https://community.ops.io/remoteimages/uploads/articles/xvp9m4zldq37np7dd0jy.png)

So this is Terraform in a nutshell. 

## Manage Kubernetes Resources via Terraform

Terraform’s Kubernetes (K8S) provider is used to interact with the resources supported by Kubernetes and offers many benefits, but it's important to note that the capability is still new. This means you might not have all of the resources available in the provider or there might be some open bugs.

That said, how would this look?
Let's look at an example for an AKS cluster in Terraform:
```hcl
resource "azurerm_kubernetes_cluster" "main" {
  name                              = "aks-${var.prefix}-${var.env}"
  location                          = azurerm_resource_group.main.location
  resource_group_name               = azurerm_resource_group.main.name
  dns_prefix                        = "${var.prefix}-${var.env}"
  role_based_access_control_enabled = var.ad_admin_group == "" ? false : true
  kubernetes_version                = var.kubernetes_version

  dynamic "azure_active_directory_role_based_access_control" {
    for_each = var.ad_admin_group == "" ? [] : [1]
    content {
      admin_group_object_ids = [var.ad_admin_group]
      azure_rbac_enabled     = true
    }
  }

  dynamic "oms_agent" {
    for_each = var.oms_agent_enabled == true ? [1] : []
    content {
      log_analytics_workspace_id = var.log_analytics_workspace_id
    }
  }
  azure_policy_enabled             = var.azure_policy_enabled
  http_application_routing_enabled = false
  api_server_authorized_ip_ranges  = var.api_server_authorized_ip_ranges

  default_node_pool {
    name                 = "default"
    enable_auto_scaling  = var.enable_auto_scaling
    max_count            = var.enable_auto_scaling ? var.max_count : null
    min_count            = var.enable_auto_scaling ? var.min_count : null
    node_count           = var.node_count
    type                 = "VirtualMachineScaleSets"
    vm_size              = var.node_size
    tags                 = var.tags
    orchestrator_version = var.node_pool_orchestrator_version
  }

  service_principal {
    client_id     = azuread_application.main.application_id
    client_secret = azuread_service_principal_password.main.value
  }

  tags = var.tags
}
```

So you create the resources, run `terraform apply` and it will provision your infrastructure.

For the **_deployment_** we create a separate Terraform file using the kubernetes deployment resource:

```hcl
resource "kubernetes_deployment" "example" {
  metadata {
    name = "example"
    labels = {
      App = "Example"
    }
  }

  spec {
    replicas = 2
    selector {
      match_labels = {
        App = "Example"
      }
    }
    template {
      metadata {
        labels = {
          App = "Example"
        }
      }
      spec {
        container {
          image = "nginx:1.7.8"
          name  = "example"

          port {
            container_port = 80
          }

          resources {
            limits = {
              cpu    = "0.5"
              memory = "512Mi"
            }
            requests = {
              cpu    = "250m"
              memory = "50Mi"
            }
          }
        }
      }
    }
  }
}

```

And in order to create this you again run `terraform apply` and confirm the changes.

Same approach will apply to creating a **_service_**:
```hcl
resource "kubernetes_service" "example" {
  metadata {
    name = "example"
  }
  spec {
    selector = {
      App = kubernetes_deployment.example.spec.0.template.0.metadata[0].labels.App
    }
    port {
      port        = 80
      target_port = 80
    }

    type = "LoadBalancer"
  }
}

```
And if you want to scale this setup then the approach is:
-  Make the changes to the replica count
```hcl
  spec {
    replicas = 3
    selector {
      match_labels = {
        App = "Example"
      }
    }
```
- Apply the terraform code and confirm the changes

Aside from this we can store the `kubernetes_namespace` resource:
```hcl
resource "kubernetes_namespace" "example" {
  metadata {
    annotations = {
      name = "example"
    }
    name = "example"
  }
}
```

And any secrets you might need:
```hcl
resource "kubernetes_secret" "example" {
  metadata {
    name      = "example"
    namespace = "example"
  }

  data = {
    "some_setting" = "false"
  }
}
```

Using this approach means you can take _advantage_ of the benefits of Terraform including:
- can use one tool for managing your infrastructure resources and also for your cluster management
- you use one language for all your infrastructure resources and also the k8s objects
- you can nicely see the plan of your changes before provisioning resources

The _disadvantages_ are of course you need to be familiar with hcl language and if the team is new adopting it would take a bit of time.

The K8s Terraform provider might not fully support all the beta objects so you might need to wait.

If you are interested in provisioning a cluster and all the K8s objects via Terraform please check the [official documentation](https://learn.hashicorp.com/tutorials/terraform/kubernetes-provider) for step by step settings.

## What is Helm?

[Helm](https://helm.sh/) is a package manager tool that helps you manage Kubernetes applications. Helm makes use of **Helm Charts** to define, install, and upgrade Kubernetes application.

Let's look over some terminology when working with Helm: 

_Helm:_ is the command-line interface that helps you define, install, and upgrade your Kubernetes application using charts.

_Charts:_ are the format for Helm’s application package. The **chart** is a bundle of information necessary to create an instance of a Kubernetes application. Basically a package of files and templates that gets converted into Kubernetes objects at deployment time.

_Chart repository:_ is the location where you can store and share packaged charts.

_The config:_ contains configuration information that can be merged into a packaged chart to create a releasable object.

_The Release:_ is a running instance of a chart, combined with a specific config. It is created by Helm to track the installation of the charts you created/defined.

For more details on [Helm architecture](https://helm.sh/docs/topics/architecture/).

Some of the _benefits_ Helm brings:

- Charts are in YAML format and are **reusable** because they provide repeatable application installation. Because of this you can use them in multiple environments (think dev, staging and prod following the same one). 
- Because charts build a repeatable process this makes deployments easier.
- A lot of charts are **already [available](https://artifacthub.io/)**, but you can create your own as well - **custom** charts.
- You can create **dependencies** between the charts and can also use [**sub-charts**](https://helm.sh/docs/chart_template_guide/subcharts_and_globals/) to add more flexibility to your setup.
- Charts serve as a single point of authority.
- Releases are tracked.
- You can upgrade or rollback multiple K8s objects together.
- Charts can be easily installed/ uninstalled.

## Manage Kubernetes Resources via Helm

We looked over the main terminology when using Helm, but let's see how it would look like.

First thing we need to do is in the repository where we have our code we run the `helm create <app_name>` command. This command creates a chart directory along with the common files and directories used in a chart. [More information on the command](https://helm.sh/docs/helm/helm_create/).

And this will create a structure as follows:
```
.
└── example
    ├── Chart.yaml
    ├── charts
    ├── templates
    │   ├── NOTES.txt
    │   ├── _helpers.tpl
    │   ├── deployment.yaml
    │   ├── hpa.yaml
    │   ├── ingress.yaml
    │   ├── service.yaml
    │   ├── serviceaccount.yaml
    │   └── tests
    │       └── test-connection.yaml
    └── values.yaml

```
We notice it created:
A _Chart.yaml_ file which just contains the information about the chart.
```yaml
apiVersion: v2
name: example
description: A Helm chart for Kubernetes

# A chart can be either an 'application' or a 'library' chart.
#
# Application charts are a collection of templates that can be packaged into versioned archives
# to be deployed.
#
# Library charts provide useful utilities or functions for the chart developer. They're included as
# a dependency of application charts to inject those utilities and functions into the rendering
# pipeline. Library charts do not define any templates and therefore cannot be deployed.
type: application

# This is the chart version. This version number should be incremented each time you make changes
# to the chart and its templates, including the app version.
# Versions are expected to follow Semantic Versioning (https://semver.org/)
version: 0.1.0

# This is the version number of the application being deployed. This version number should be
# incremented each time you make changes to the application. Versions are not expected to
# follow Semantic Versioning. They should reflect the version the application is using.
# It is recommended to use it with quotes.
appVersion: "1.16.0"

```

The _charts_ directory where you can add any charts that your chart depends on.

The _templates_ directory:
A directory to store partials and helpers. The file called **_helpers.tpl** is the default location for template partials that the rest of the yaml files rely on as we will see.

How does this work?

Let's take a small example, in that file we define the fullname:

```yaml
{{/*
Create a default fully qualified app name.
We truncate at 63 chars because some Kubernetes name fields are limited to this (by the DNS naming spec).
If release name contains chart name it will be used as a full name.
*/}}
{{- define "example.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}
```

And in our `service.yaml` file we can include the defined fullname like:

```yaml
apiVersion: v1
kind: Service
metadata:
  name: {{ include "example.fullname" . }}
  labels:
    {{- include "example.labels" . | nindent 4 }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: http
      protocol: TCP
      name: http
  selector:
    {{- include "example.selectorLabels" . | nindent 4 }}
```

And in the `values.yaml` file we can also override it:
```yaml
# Default values for example.
# This is a YAML-formatted file.
# Declare variables to be passed into your templates.

replicaCount: 1

image:
  repository: nginx
  pullPolicy: IfNotPresent
  # Overrides the image tag whose default is the chart appVersion.
  tag: ""

imagePullSecrets: []
nameOverride: ""
fullnameOverride: ""

serviceAccount:
  # Specifies whether a service account should be created
  create: true
  # Annotations to add to the service account
  annotations: {}
  # The name of the service account to use.
  # If not set and create is true, a name is generated using the fullname template
  name: ""

podAnnotations: {}

podSecurityContext: {}
  # fsGroup: 2000

securityContext: {}
  # capabilities:
  #   drop:
  #   - ALL
  # readOnlyRootFilesystem: true
  # runAsNonRoot: true
  # runAsUser: 1000

service:
  type: ClusterIP
  port: 80
(...)
```

In the example above the fullname will be the Chart name since we didn't pass any value in the `values.yaml` file for the override based on the definition of the parameter in the `_helpers.tpl` file.

The _values.yaml_ file which contains the default values for your templates. You can at this point split the values file per each environment: one for staging, one for production etc.

From here onwards you can start configuring based on what you need. If you check the yaml files you will notice the structure is very similar to what we defined in the K8s objects terraform code.

For the installation of the charts you can either go one by one and make use of the `helm install/upgrade` commands OR you can add this in your CI/CD pipelines.

Going by the command line could look something like:
`helm upgrade example infra/charts/example --install --wait --atomic --namespace=example --set=app.name=example --values=infra/charts/example/values.yaml`
where:
- infra/charts/example - is the location of your `Chart.yaml` file
- values=infra/charts/example/values.yaml - is the location of the values file
- `--wait` - will wait until either the release is successful or the default timeout is reached (5m) if no timeout is specified
- `--atomic`- if set, upgrade process rolls back changes made in case of failed upgrade

Check the full synopsis of the command [here](https://helm.sh/docs/helm/helm_upgrade/).

_helm install or helm upgrade --install?_

The _install_ sub-command always installs a brand new chart, while the _upgrade_ sub-command can upgrade an existing chart and install a new one, if the chart hasn't been installed before. 
>For simplicity you can always you the upgrade sub-command.

## Using Helm and Terraform Together
Helm and Terraform are not mutually exclusive and can be used together in the same K8s setup even if the actual setup really depends on your project complexity, which benefits you want to make use of and which drawbacks you can live with.

In a potential setup where you would use both you could structure it something like:

- use Terraform to _create and manage resources_: the K8s Cluster, the K8s namespace, and the K8s secrets( if any )
- use Helm charts to _deploy_ your applications

This is the setup we currently use and it has served us well so far.

It is worth mentioning that you can also use Terraform to handle your Helm deploys using the `helm_release` [resource](https://registry.terraform.io/providers/hashicorp/helm/latest/docs/resources/release). 

In this approach you would have both infrastructure and provisioning in one place - in Terraform. I will not go in this post in the differences between them, but I will mention going with this approach should depend on how _frequent_ you need to apply changes to your infrastructure because the way this works is during `terraform apply` operation the helm release will take place.

There is no one-size-fits-all approach, but you should tailor tooling and the strategy to your needs.

_Hope you find this helpful. Thank you for reading and feel free to comment on your experience and what you prefer to use and why._
