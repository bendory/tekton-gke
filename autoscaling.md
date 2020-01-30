# Autoscaling Node Pools

Docs:
* https://cloud.google.com/kubernetes-engine/docs/concepts/cluster-autoscaler

# Advantages of Autoscaling

Autoscaling nodes gives you an elastic supply of compute resources, allowing you to cut costs by freeing resources you aren't using.

For example, if your workloads tend to happen during the day (e.g., running CI on pushes to your source repo), you can scale up your compute resources to run those workloads while engineers are making changes during the day, then scale them back down overnight when there's less work to do.

# Caveats

## Scaling Down

When a node is scaled down due to reduced load, workloads on that node are given a ten minute grace period to finish. Because TaskRuns can be long-running, they might be disrupted by these nodes scaling down.

## Ensuring Minimum Resources

You should set a reasonable minimum node count, in case resources aren't available in the zone when your cluster wants to scale up. You can achieve this by setting `--min-nodes`, or by creating a separate [node pool](https://cloud.google.com/kubernetes-engine/docs/concepts/node-pools) which autoscales up and down, alongside a non-autoscaling node pool that you're guaranteed to have available.

You should keep an eye on the number of nodes you normally use at off-peak times, even if that's higher than your cluster's autoscaling minimum.

# Enabling Autoscaling

When creating a cluster:

```
gcloud beta container clusters create [CLUSTER NAME] \
    --zone=[ZONE] \
    --enable-autoscaling --min-nodes=[MINIMUM] --max-nodes=[MAXIMUM] \
    --machine-type=[MACHINE TYPE]
```

This creates a cluster that will have a minimum number of nodes available, and will scale up in response to load up to the configured max.
    
## Disabling Autoscaling

If for some reason autoscaling does not meet your needs after all, you can simply disable autoscaling and configure the constant number of nodes you need available.

```
gcloud beta container clusters update [CLUSTER NAME] \
    --zone=[ZONE] \
    --no-enable-autoscaling
    --num-nodes=[NODES]
```

This updates the cluster to disable autoscaling, and ensure a constant number of nodes in the pool.
