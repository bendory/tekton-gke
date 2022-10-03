# Image Streaming

Image streaming enables GKE to stream image data as needed; rather than pulling
the entire image up front, GKE retrieves bytes of the image as needed.  This
allows workloads to initialize without waiting for the entire image to download
which leads to improvements in initialization times.  In the worst case (where
every byte in the image is needed), you experience a lower latency start
(because fewer bytes are pulled up front) while completion time is unchanged
(the entire image is eventually pulled). In the best case (where only parts of
the image are needed), substantial pull and unpacking time are saved by only
pulling the needed parts of the image.

Docs:
* https://cloud.google.com/kubernetes-engine/docs/how-to/image-streaming

## Setting up Image Streamining on your cluster

Note that you must use [Container-Optimized OS with
containerd](https://cloud.google.com/kubernetes-engine/docs/concepts/using-containerd?hl=en)
and reference images in [Artifact
Registry](https://cloud.google.com/artifact-registry) to benefit from image
streaming. (There are additional
[limitations](https://cloud.google.com/kubernetes-engine/docs/how-to/image-streaming#limitations).)

```
gcloud container clusters create $CLUSTER_NAME \
    --image-type="COS_CONTAINERD" --enable-image-streaming ...
```

To enable image streaming on an existing cluster:

`gcloud container clusters update $CLUSTER_NAME --enable-image-streaming`

Note that you can also enable (or disable) image streaming on a per-node pool
basis.

## Disabling Image Streaming

`gcloud container clusters update $CLUSTER_NAME --no-enable-image-streaming`

