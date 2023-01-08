---
title: "K8s - How to restart Kubernetes Pods"
date: 2022-05-25T12:35:04+01:00
draft: false
tags: ["kubernetes"]
---
This article applies for **Kubernetes v1.15** and above.

[Kubernetes](https://kubernetes.io/), also known as K8s, is an open-source system for automating deployment, scaling, and management of containerized applications.

It groups containers that make up an application into logical units for easy management and discovery. But what if something happens to the container? In this case, you might need a quick and easy way to restart it.

[Kubernetes Pods](https://kubernetes.io/docs/concepts/workloads/pods/pod-lifecycle/) usually run until there is a new deployment that will replace them. Therefore, there is no straightforward way to restart a single pod.

What happens when one container fails is that, instead of restarting it, it will be replaced. 

## Restarting Pods Options

There are a few available options that we will cover in this article:

1. Scaling down the number of replicas indicating how many Pods it should be maintaining in the ReplicaSet, effectively removing pods, then scaling back up
      **Causes downtime**

2. Deleting a single pod, forcing K8s to recreate it 
     **Might cause downtime**

3. Starting a rollout (rolling restart method)
     **No downtime**

So let's look at each option in a bit of details and keep in mind that each option could work for you depending on your needs. Some questions to ask: is it a live environment? is it a new setup? can you afford and outage on the app?

## 1. Changing Replicas

An option for restarting the pods is to effectively "shut them off" by scaling the number of deployment replicas first to zero:

```
kubectl scale deployment <name> --replicas=0
```
In this case, K8s will remove all the replicas that are no longer required.

And then, scaling them back up to the desired number.

```
kubectl scale deployment <name> --replicas=<desired_number>
```

This will stop and terminate all current pods and will schedule new pods in their place.

Because we are "shutting down" pods, this option will cause downtime since there will be no container available. So if you're running on a production system the rolling restart method would be the better approach.

The names of the new scheduled pods _will_ be different from the previous ones. 

Run the following command to get the new names of the pods:
```
kubectl get pods -n <namespace>
```
## 2. Deleting a Pod

First we get all the pods in a namespace by running the following command:
```
kubectl get pods -n <namespace>
```

Then we delete a single pod by running the following command:
```
kubectl delete pod <pod_name> -n <namespace>
```

K8s will note the change and the state difference and will schedule new pods until the desired state is achieved.

## 3. Rolling Restart

From version 1.15 K8s now allows you to execute a rolling restart of your deployment. 
_Note: Not only the kubectl versions needs to be updated, but make sure the cluster is running on this version as well._

Rolling restart is used to restart all the pods from a deployment in sequence by running the following command: 

```
kubectl rollout restart deployment -n <yournamespace>
```

After running this command, K8s proceeds to shut down and restart each pod in the deployment one by one. 

Because this is done sequentially, there is always some pod running meaning the application itself will still be available, effectively allowing for **zero downtime**.

_Thank you for reading and I hope this helps someone!_
