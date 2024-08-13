---
title: "AWS: Handling 'Cannot delete entity, must remove tokens from principal first' error"
description: "How to handle deleting IAM users in AWS when you are asked to delete the token from the principal first."
date: 2024-02-07T13:32:57+01:00
draft: false
tags: ["aws", "iam", "terraform"]
---

This blog post will be a quick one focusing on troubleshooting a less clear error, _'Cannot delete entity, must remove tokens from principal first'_, that Terraform can throw when you try to delete IAM users from AWS.

Let's assume that in your Terraform configuration, you manage IAM users and you want to delete one of them. You'd think that by simply removing the Terraform code and then running `terraform apply` it will delete the users. Which was my case. But then as soon as I ran the command to destroy the resource I ran into an issue:

```console
Terraform used the selected providers to generate the following execution plan. Resource actions are indicated with the following symbols:
  - destroy

Terraform will perform the following actions:

  # aws_iam_user.little_tester will be destroyed
  # (because aws_iam_user.little_tester is not in configuration)
  - resource "aws_iam_user" "little_tester" {
      - arn           = "arn:aws:iam::xxxxxxxxxx:user/little_tester" -> null
      - force_destroy = false -> null
      - id            = "little_tester" -> null
      - name          = "little_tester" -> null
      - path          = "/" -> null
      - tags          = {
          - "Company"  = "MyCompany"
          - "Location" = "Aruba"
          - "Unit"     = "Front Desk"
        } -> null
      - tags_all      = {
          - "Company"  = "MyCompany"
          - "Location" = "Aruba"
          - "Unit"     = "Front Desk"
        } -> null
      - unique_id     = "AAAAAAAAAAAAAAAAA" -> null
    }

Plan: 0 to add, 0 to change, 1 to destroy.

Do you want to perform these actions?
  Terraform will perform the actions described above.
  Only 'yes' will be accepted to approve.

  Enter a value: yes

aws_iam_user.little_tester: Destroying... [id=little_tester]
╷
│ Error: deleting IAM User (little_tester): DeleteConflict: Cannot delete entity, must remove tokens from principal first.
│  status code: 409, request id: ...
│
```

## So what does this mean?

The error `Cannot delete entity, must remove tokens from principal first.` says that the user has some tokens that need to be removed before the user itself can be deleted. The tokens it refers to can be active access keys or registered MFA devices.

The decision to prevent the deletion of a user if any of these active tokens are associated with it makes sense from a security perspective because it aims to prevent the accidental deletion of users that are still active.

A way to confirm if this is the case is to go to AWS Console and check the user's Security credentials. There you should see any active access keys or registered MFA devices.

Having checked that, I saw that the user had an Access key that was still active and had an active MFA device. I removed both manually and then ran `terraform apply` again. And it worked! The user was deleted successfully.

## How can this happen?

The user's access token and the MFA device configured to his account were not managed by Terraform, meaning they were created manually. So Terraform was not aware of them and could not delete them. This was preventing the deletion of the user.

How this could come to be is if the user was created through Terraform code, but all the other configurations were done manually after the user was created: adding an access key, adding an MFA device, etc. So then you end up with a mix of Terraform-managed and non-Terraform-managed resources.

Something to think about for future cases, this could also happen if you create a user group in Terraform and then add users to it manually later on. These users will be part of the group, but Terraform will not be aware of them and will not be able to manage them. Or in any other scenario where you mix non-Terraform-managed and Terraform-managed resources.

## What can you do?

The first option is to add the access key and MFA device to the Terraform configuration so the creation and removal of the users will be part of a complete flow fully managed by Terraform.

The second option is to simply manually go to AWS Console > IAM, and check the user's Security credentials and MFA devices. For the active ones simply deactivate them and remove them manually. Then simply run to your configuration and run `terraform apply` again.

And lastly, you can add the [`force_destroy` argument](https://registry.terraform.io/providers/hashicorp/aws/latest/docs/resources/iam_user#force_destroy) to the `aws_iam_user` resource in your Terraform configuration.

> _force_destroy - (Optional, default false) When destroying this user, destroy even if it has non-Terraform-managed IAM access keys, login profile or MFA devices. Without force_destroy a user with non-Terraform-managed access keys and login profile will fail to be destroyed._

By enabling it, it will allow Terraform to delete the user even if it has non-Terraform-managed access keys and MFA devices.

**Warning!**

While it does seem a convenient option, be very careful with this argument, as it can lead to the accidental deletion of users that are still active. So I would advise you to use it only if you are sure that the user is not active (maybe have a check in place that runs before the destruction of the resources), that you are aware of the security implications and lastly check the access of the team members that can run the Terraform code.

## Conclusion

If Terraform cannot delete your AWS IAM users remember to check the user's Security credentials and look for any active access keys or MFA devices. If they are active, deactivate and remove them. How you handle it in your Terraform code is up to you, but remember to be careful with the `force_destroy` option.

_Hope this helps someone out there!_
