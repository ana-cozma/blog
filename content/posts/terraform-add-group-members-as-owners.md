---
title: "Terraform: Add Group Members as Owners to a Service Principal or Az AD Application"
date: 2023-04-21T11:18:28+02:00
draft: false
tags: ["terraform", "azure"]
---

Currently, there is no option for adding a Group as an Owner to a Service Principal or Azure AD Application.

If you try you will most likely run into the following error:

```bash
unexpected status 400 with OData error: Request_BadRequest: The reference target
│ 'Group_0000000-0000-000000-000000' of type 'Group' is invalid for the 'owners' reference.
```

This is a quick post on **how you can add members of an Az AD Group as Owners of a Service Principal or Azure AD Application**. This is useful if you want to give your team members access to the Service Principal or Azure AD Application without giving them access to the Azure Subscription.

It's also useful if you want to manage ownership of the Service Principal or Azure AD Application dynamically. For example, if a team member leaves or joins the company you can just add or remove them from the group and this will automatically update the ownership of the Service Principal or Azure AD Application.

The Terraform code below assumes you already have the group created in Azure AD.

```hcl
## Adding the current subscription and Azure AD client data sources
data "azurerm_subscription" "current" {}

data "azuread_client_config" "current" {}

## Adding the group data source of the members we want to retrieve
data "azuread_group" "group_name" {
  display_name     = "Organization.GroupName"
}

## Retrieve the list of group member ids
locals {
  group_member_object_ids = toset(concat([data.azuread_client_config.current.object_id],data.azuread_group.group_name.members))
}

## Adding the group member object ids to the owners of the application
resource "azuread_application" "application" {
  display_name = "${var.name}-app"
  owners       = local.group_member_object_ids

  app_role {
    allowed_member_types = ["User", "Application"]
    (...)
  }
}

## Adding the group member object ids to the owners of the service principal
resource "azuread_service_principal" "service_principal" {
  application_id = azuread_application.application.application_id
  owners         = local.group_member_object_ids
}
```

Hope this helps someone out there!
