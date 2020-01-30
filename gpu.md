# Tekton and GPUs

1. Create a cluster with GPUs:
    * Create an autoscaling GPU node pool: https://cloud.google.com/kubernetes-engine/docs/how-to/gpus#gpu_pool
1. Install nvidia drivers: https://cloud.google.com/kubernetes-engine/docs/how-to/gpus#installing_drivers
1. Install Tekton and `tkn`
1. Define and run a Task that prints information about available GPUs:

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

Then, run the Task:

```yaml
$ tkn task start -f gpu.yaml
Taskrun started: gpu-run-cb6ws
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

This demonstrates that GPUs are attached and available to the TaskRun execution environment. Instead of running the nvidia/cuda image, you could run your own image that takes advantage of available GPUs.
