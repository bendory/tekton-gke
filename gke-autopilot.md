# GKE Autopilot

Docs:
- https://cloud.google.com/kubernetes-engine/docs/concepts/autopilot-overview
- https://cloud.google.com/kubernetes-engine/docs/how-to/autopilot-spot-pods

# Advantages of GKE Autopilot

GKE Autopilot aims to remove some of the maintenance burden of operating a cluster, in exchange for removing some configuration options and setting sane and secure defaults.

In GKE Autopilot, Nodes are not accessible to cluster users, and usage is billed based on Pod usage, not node availability.

GKE Autopilot also enables "Spot Pods", which are like [preemptible VMs](./preemptible-vms.md), but at the Pod level instead of the Node level.
Spot Pods are billed at a 60-91% discount compared to normal Pods.
At this time, for example, Spot Pods are $0.01335/CPU/hour, compared to the normal GKE Autopilot price of $0.0445/CPU/hour, a 70% discount.

# Using GKE Autopilot with Tekton

At this time, you can't seem to create Autopilot clusters using `gcloud`, you have to use the Cloud Console instead.

In order for Tekton's webhook validation to work correctly, you need to create a GKE Autopilot cluster at version v1.21.4 or later.
At this time, this means selecting the **Rapid** release channel, instead of the default **Stable** release channel.

After you create a GKE Autopilot cluster at version v1.21.4 or later, you can [install Tekton](https://tekton.dev/docs/pipelines/install/) in the normal manner.

## Using Spot Pods

Pods can be scheduled as Spot Pods using a node selector:

```
nodeSelector:
  cloud.google.com/gke-spot: "true"
```

You can set this in a TaskRun's `podTemplate`:

```
apiVersion: tekton.dev/v1beta1
kind: TaskRun
metadata:
  name: mytaskrun
spec:
  taskRef:
    name: mytask
  podTemplate:
    nodeSelector:
      cloud.google.com/gke-spot: "true"
```

Or for all TaskRuns in a PipelineRun:

```
apiVersion: tekton.dev/v1beta1
kind: PipelineRun
metadata:
  name: mypipelinerun
spec:
  pipelineRef:
    name: mypipeline
  podTemplate:
    nodeSelector:
      cloud.google.com/gke-spot: "true"
```

Or you can make this the default for all TaskRuns in the cluster, by updating `config-defaults`:

```
$ kubectl edit configmap config-defaults --namespace tekton-pipelines
```

```
apiVersion: v1
kind: ConfigMap
metadata:
  name: config-defaults
  namespace: tekton-pipelines
data:
  default-pod-template: |
    nodeSelector:
      cloud.google.com/gke-spot: "true"
```
