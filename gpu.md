# Tekton and GPUs

Docs:

* https://cloud.google.com/kubernetes-engine/docs/how-to/gpus

## Setting up GPUs on your cluster

Create a node pool with GPUs available:

```
gcloud container node-pools create gpu-pool \
    --cluster=[CLUSTER-NAME] \
    --accelerator=type=[GPU-TYPE],count=[NUMBER] \
    --num-nodes=[NUM-NODES]
```

For example, `--accelerator=type=nvidia-tesla-p4,count=1` will make a node pool with one Tesla P4 GPU available per node.

Install necessary nvidia drivers:

```
kubectl apply -f https://raw.githubusercontent.com/GoogleCloudPlatform/container-engine-accelerators/master/nvidia-driver-installer/cos/daemonset-preloaded.yaml
```

## Run a Task with GPUs

Now that the node pool is created, you can run a TaskRun and tell the Kubernetes scheduler to make sure it runs on a node with GPUs available.

This Task simply prints information about available GPUs:

```yaml
apiVersion: tekton.dev/v1alpha1
kind: Task
metadata:
  name: gpu
spec:
  steps:
  - image: nvidia/cuda:10.0-runtime-ubuntu18.04
    script: nvidia-smi
    resources:
      limits:
       nvidia.com/gpu: 1
```

Run this Task using `tkn`:

```yaml
$ tkn task start -f gpu.yaml
Taskrun started: gpu-cb6ws
Waiting for logs to be available...
[unnamed-0] + nvidia-smi
[unnamed-0] Tue Jan  7 16:11:57 2020       
[unnamed-0] +-----------------------------------------------------------------------------+
[unnamed-0] | NVIDIA-SMI 418.67       Driver Version: 418.67       CUDA Version: 10.1     |
[unnamed-0] |-------------------------------+----------------------+----------------------+
[unnamed-0] | GPU  Name        Persistence-M| Bus-Id        Disp.A | Volatile Uncorr. ECC |
[unnamed-0] | Fan  Temp  Perf  Pwr:Usage/Cap|         Memory-Usage | GPU-Util  Compute M. |
[unnamed-0] |===============================+======================+======================|
[unnamed-0] |   0  Tesla P4            Off  | 00000000:00:04.0 Off |                    0 |
[unnamed-0] | N/A   32C    P8     7W /  75W |      0MiB /  7611MiB |      0%      Default |
[unnamed-0] +-------------------------------+----------------------+----------------------+
[unnamed-0]                                                                                
[unnamed-0] +-----------------------------------------------------------------------------+
[unnamed-0] | Processes:                                                       GPU Memory |
[unnamed-0] |  GPU       PID   Type   Process name                             Usage      |
[unnamed-0] |=============================================================================|
[unnamed-0] |  No running processes found                                                 |
[unnamed-0] +-----------------------------------------------------------------------------+
```

This demonstrates that GPUs are attached and available to the TaskRun execution environment. Instead of running the `nvidia/cuda` image, you could run your own image that takes advantage of available GPUs to train an ML model.
