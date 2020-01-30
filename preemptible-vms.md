# Preemptible VMs

Docs:

* https://cloud.google.com/kubernetes-engine/docs/how-to/preemptible-vms
* https://cloud.google.com/compute/docs/instances/preemptible
* https://cloud.google.com/compute/docs/instances/create-start-preemptible-instance#best_practices
* https://cloud.google.com/kubernetes-engine/docs/node-taints

## Advantages of Preemptible VMs

Preemptible VMs cost **~80% less than** regular VMs. If you are comfortable with the limitations, using preemptible VMs can save a lot on costs.

## Caveats

### Node preemption and retries

TaskRuns that are running on a node when it is preempted are forcibly cancelled. However, TaskRuns that are run as part of a PipelineRun can be configured to retry in that case up to a specified number of times. If you specify that they should be retried, make sure they are idempotent; that is, that retrying the workload won't have harmful side effects like sending notifications or billing customers.

Preemptible VMs last at most 24 hours before being terminated. Any TaskRun that runs on a preemptible VM should be expected to finish within 24 hours at most, and it can be helpful to enforce this with TaskRun's `timeout` field.

It is recommended that workloads running in Preemptible VMs should periodically checkpoint their progress so that they don't have to start long processes from the beginning when they get preempted. In Tekton Pipelines, this means that PipelineRuns should consist of many relatively short-lived TaskRuns. When a TaskRun is preempted, it will be retried instead of having to retry the whole PipelineRun from the beginning.

### Node unavailability

If you use a node pool of preemptible VMs, there is no guarantee that _any_ of those nodes will be available at a given time. For this reason, it's recommended that the Tekton controller components run on a smaller node pool of regular, non-preemptible VMs. This will ensure that new Tekton resources can be created and enqueued until preemptible VMs become available to run workloads.

Depending on your use case, you might want to run some small number of non-preemptible VMs, so that there are always guaranteed resources available to run your Tekton workloads.

Preemptible VMs might be unavailable in your preferred zone for hours or even days at a time. Don't run workloads that must complete within a specific time. Preemptible VMs might be unavailable during times when VMs of all types are unavailable in your zone, so you shouldn't, for instance, rely on an autoscaling pool of non-preemptible VMs to pick up slack while your preemptible VMs are unavailable.

Preemptible VMs are made out of excess resources, so it's often easier to get smaller instances during peak usage times. Size your preemptible node pool to prefer smaller machine types (n1-standard-2 or n1-standard-4) over large machine types (n1-standard-48 or n1-standard-64) to ensure resources are available

Selecting smaller machine types can also limit the impact of a preemption, since fewer TaskRuns will be running on those nodes when they get preempted.

## Enabling Preemptpible VMs

To create a cluster with a small number of non-preemptible nodes and a larger autoscaling pool of preemptible nodes, run this script:

```
gcloud beta container clusters create [CLUSTER NAME] \
    --zone=[ZONE] \
    --num-nodes=3 \
    --machine-type=[MACHINE TYPE]
gcloud beta container node-pools create preemptible-pool \
    --preemptible \
    --cluster [CLUSTER NAME] \
    --zone=[ZONE] \
    --machine-type=[MACHINE TYPE] \
    --enable-autoscaling --min-nodes=0 --max-nodes=50 \
    --node-taints=cloud.google.com/gke-preemptible="true":NoSchedule
```
    
### Deleting the Preemptible Node Pool

If for some reason preemptible VMs do not meet your needs after all, you can simply remove the node pool of preemptible VMs and increase the size of the default node pool.

```
gcloud beta container node-pools delete preemptible-pool \
    --cluster [CLUSTER NAME] \
    --zone=[ZONE]
```

And, to increase the size of the default (non-preemptible) node pool:

```
gcloud beta container clusters update [CLUSTER NAME] \
    --zone=[ZONE] \
    --min-nodes=[NEW POOL SIZE]
```

## Running TaskRuns on Preemptible VMs

To declare that a TaskRun must run on a preemptible node, define a NodeSelector in the TaskRun's PodTemplate:

```
apiVersion: tekton.dev/v1alpha1
kind: TaskRun
metadata:
  name: preemptible-taskrun
spec:
  …
  timeout: 24h
  podTemplate:
    nodeSelector:
      cloud.google.com/gke-preemptible: "true"
```

### Avoiding Preemptible VMs

For TaskRuns that are not resilient to or tolerant of preemption, it can be helpful to tell the scheduler not to run a TaskRun on a preemptible VM.

The preemptible-pool node pool above is configured to apply a node taint to its nodes, which allows Pods and TaskRuns to require the scheduler not to run them on the preemptible node pool's VMs.

```
apiVersion: tekton.dev/v1alpha1
kind: TaskRun
metadata:
  name: non-preemptible-taskrun
spec:
  …
  timeout: 24h
  podTemplate:
    tolerations:
    - key: cloud.google.com/gke-preemptible
      operator: Equal
      value: "true"
      effect: NoSchedule
```
