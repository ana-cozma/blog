---
title: "K8s: Fix Helm release failing with an upgrade still in progress"
description: "How to fix Helm release failing when Upgrade is marked as still In Progress and will not complete"
date: 2022-05-30T17:49:48+01:00
draft: false
tags: ["kubernetes", "helm"]
---
This article applies to: **Helm v3.8.0**

> _Helm helps you manage Kubernetes applications — Helm Charts help you define, install, and upgrade even the most complex Kubernetes application. More details on Helm and the commands can be found in the [official documentation](https://helm.sh/)._

Assuming you use Helm to handle your releases, you might end up in a case where the release will be stuck in a pending state and all subsequent releases will keep failing.

This could happen if:

- you run the upgrade command from the cli and accidentally (or not) interrupt it or
- you have two deploys running at the same time (maybe in Github Actions, for example)

Basically, any interruption that occurred during your install/upgrade process **could** lead you to a state where you cannot install another release anymore.

In the release logs the failing upgrade will show an error similar to the following:

```bash
Error: UPGRADE FAILED: release <name> failed, and has been rolled back due to atomic being set: timed out waiting for the condition
Error: Error: The process '/usr/bin/helm3' failed with exit code 1
Error: The process '/usr/bin/helm3' failed with exit code 1
```

The status will be stuck in: `PENDING_INSTALL` or `PENDING_UPGRADE` depending on the command you were running.

Because of this pending state when you run the command to list all releases, it will return empty:

```bash
> helm list --all                                                                               
NAME	NAMESPACE	REVISION	UPDATED	STATUS	CHART	APP VERSION
```

So what can we do now? In this article, we will look over the two options described below. Keep in mind that based on your setup it could be another issue, but I'm hoping that these two pointers will give you a place to start.

1. Roll back to the previous working version using the `helm rollback` command.
2. Delete the helm secret associated with the release and re-run the upgrade command.

So let's look at each option in detail.

## Options

### Roll back to the previous working version using the `helm rollback` command

From the official documentation:

> _This command rolls back a release to a previous revision.
The first argument of the rollback command is the name of a release, and the second is a revision (version) number. If this argument is omitted, it will roll back to the previous release._

So, in this case, let's get the history of the releases:

```bash
helm history <releasename> -n <namespace>
```

In the output, you will notice the STATUS of your release with `pending-upgrade`:

```bash
REVISION UPDATED                  STATUS          CHART     APP VERSION DESCRIPTION
1        Wed May 25 11:45:40 2022 DEPLOYED        api-0.1.0 1.16.0      Upgrade complete
2        Mon May 30 14:32:46 2022 PENDING_UPGRADE api-0.1.0 1.16.0      Preparing upgrade
```

Now let's perform the rollback by running the following command:

```bash
helm rollback <release> <revision> --namespace <namespace>
```

So in our case, we run:

```bash
> helm rollback api 1 --namespace api
Rollback was a success.
```

After we get confirmation that the rollback was successful, we run the command to get the history again.

We now see we have two releases and that our rollback was successful having the STATUS `deployed``

```bash
> helm history api -n api

REVISION UPDATED                  STATUS      CHART     APP VERSION DESCRIPTION
1        Wed May 25 11:45:40 2022 SUPERSEEDED api-0.1.0 1.16.0      Upgrade complete
2        Mon May 30 14:32:46 2022 SUPERSEEDED api-0.1.0 1.16.0      Preparing upgrade
3        Mon May 30 14:45:46 2022 DEPLOYED    api-0.1.0 1.16.0      Rollback to 1
```

So what if the solution above didn't work?

### Delete the helm secret associated with the release and re-run the upgrade command

First, we get all the secrets for the namespace by running:

```bash
kubectl get secrets -n <namespace>
```

In the output, you will notice a list of secrets in the following format:

```bash
NAME                        TYPE               DATA AGE
api                         Opaque             21   473d
sh.helm.release.v1.api.v648 helm.sh/release.v1 1    6d5h
sh.helm.release.v1.api.v649 helm.sh/release.v1 1    5d1h
sh.helm.release.v1.api.v650 helm.sh/release.v1 1    57m
```

**So what's in a secret?**

Helm3 makes use of the Kubernetes [Secrets](https://kubernetes.io/docs/concepts/configuration/secret/) object to store any information regarding a release. These secrets are basically used by Helm to store and read its state every time we run: `helm list`, `helm history` or `helm upgrade` in our case.

The naming of the secrets is **unique per namespace**. 
The format follows the following convention:
`sh.helm.release.v1.<release_name>.<release_version>`.

There is a max of 10 secrets that are stored by default, but you can modify this by setting the `--history-max ` flag in your [helm upgrade](https://helm.sh/docs/helm/helm_upgrade/) command.

>  _--history-max int                            limit the maximum number of revisions saved per release. Use 0 for no limit (default 10)_

Now that we know what these secrets are used for, let's delete the helm secret associated with the pending release by running the following command:

```bash
kubectl delete secret sh.helm.release.v1.<release_name>.v<release_version> -n <namespace>
```

Finally, we re-run the helm upgrade command (either from the command line or from your deployment workflow), which, if all is good so far, should succeed.

There is an open issue with [Helm](https://github.com/helm/helm/issues/4558) so hopefully these workaround won't be needed anymore. But it's open since 2018.

_Of course, there could be other cases or issues, but I hope this is a nice place to start. If you ran into something similar I would love to read your input on what the issue was and how you solved it especially since I didn't find the error message to be intuitive._

_Thank you for reading!_
