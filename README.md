# Running NVIDIA Isaac Lab for Reinforcement Learning on Kubernetes with Weights & Biases Tracking

This guide explains how to run distributed NVIDIA Isaac Lab training on
a Kubernetes GPU cluster using the provided StatefulSet YAML
configuration. We are using a L40 (2 nodes x 4 GPU cluster from Coreweave here, please update the hardware configs to match your cluster)

This setup supports:

-   Multi-node distributed training (2 nodes × 4 GPUs each)
-   Headless Isaac Sim execution
-   Automatic Weights & Biases (W&B) logging
-   Artifact tracking
-   Checkpoint and video upload after training
-   Persistent shared storage for logs and models

------------------------------------------------------------------------

# Documentation References

-   Isaac Lab GitHub\
    https://github.com/isaac-sim/IsaacLab

-   Isaac Sim Documentation\
    https://docs.omniverse.nvidia.com/isaacsim/latest/index.html

-   Weights & Biases Documentation\
    https://docs.wandb.ai

-   Kubernetes Documentation\
    https://kubernetes.io/docs/home/

-   NVIDIA NGC (Container Registry)\
    https://ngc.nvidia.com

------------------------------------------------------------------------

# Architecture Overview

The provided YAML deploys:

-   A StatefulSet with 2 replicas\
-   4 GPUs per pod\
-   torch.distributed multi-node training\
-   Headless Isaac Sim\
-   A 400Gi PersistentVolumeClaim (ReadWriteMany)\
-   Automatic W&B artifact download before training\
-   Automatic checkpoint + video upload after training

Training task:

    Isaac-Dexsuite-Kuka-Allegro-Lift-v0

    Other avaialble tasks and enviroments that ship with Isaac Sim: https://isaac-sim.github.io/IsaacLab/main/source/overview/environments.html

------------------------------------------------------------------------

# 1. Prerequisites

## Select the Correct Kubernetes Context for your cluster

``` bash
kubectl config get-contexts
kubectl config use-context <your-cluster-context>
```

------------------------------------------------------------------------

# 2. NVIDIA Container Registry (NGC) Setup

Container image used:

    nvcr.io/nvidia/isaac-lab:2.3.2

Create an NGC account:

https://ngc.nvidia.com

Generate an NGC API key and create a Kubernetes image pull secret:

``` bash
kubectl create secret docker-registry nvcr-secret   --docker-server=nvcr.io   --docker-username='$oauthtoken'   --docker-password='<YOUR_NGC_API_KEY>'  
```

If required, add to your pod spec:

``` yaml
imagePullSecrets:
  - name: nvcr-secret
```

------------------------------------------------------------------------

# 3. Weights & Biases Setup

Create a W&B account:

https://wandb.ai

Generate an API key:

https://wandb.ai/authorize

Store it as a Kubernetes secret:

``` bash
kubectl create secret generic wandb-api-key   --from-literal=WANDB_API_KEY=<YOUR_WANDB_API_KEY>
```

Referenced in YAML as:

``` yaml
- name: WANDB_API_KEY
  valueFrom:
    secretKeyRef:
      name: wandb-api-key
      key: WANDB_API_KEY
```

------------------------------------------------------------------------

# 4. Running Training

Apply your YAML:

``` bash
kubectl apply -f isaaclab-max.yaml
```

Check pods:

``` bash
kubectl get pods
```

View logs:

``` bash
kubectl logs -f isaaclab-max-workers-0
kubectl logs -f isaaclab-max-workers-1
```

------------------------------------------------------------------------

# 5. Cleanup

Delete deployment:

``` bash
kubectl delete -f isaaclab-max.yaml
```

Delete PVC (optional):

``` bash
kubectl delete pvc isaaclab-max-logs-pvc
```

------------------------------------------------------------------------

# Summary

Workflow:

1.  Select cluster context\
2.  Ensure NGC secret exists\
3.  Ensure W&B secret exists\
4.  Apply YAML\
5.  Monitor logs

Training runs distributed across 8 GPUs. Artifacts download
automatically. Checkpoints and videos upload automatically to Weights &
Biases.
