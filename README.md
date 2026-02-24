# Running Nvidia Isaac Sim 

# Initial Setup on Coreweave Cluster
### 1. Get Isaac Lab
```
git clone https://github.com/isaac-sim/IsaacLab.git
cd IsaacLab
```
### 2.  Accept EULAs so the container can run headless without prompts
#### Isaac Sim/Omniverse require EULA acceptance in non-interactive runs.
```
export OMNI_KIT_ACCEPT_EULA=YES
export ACCEPT_EULA=Y
export PRIVACY_CONSENT=Y
```
### 3. Create docker/cluster/.env.cluster

```
# Isaac Lab cluster interface 
export CLUSTER_TYPE=slurm
export CLUSTER_DEFAULT_PARTITION=h100
# Where SIF will live and where to copy logs/checkpoints
export CLUSTER_REMOTE_ROOT=/mnt/home/$USER/isaaclab_cluster
export CLUSTER_LOGS_DIR=/mnt/home/$USER/isaaclab_logs
# Build profile (the default "base" image is fine)
export ISAACLAB_DOCKER_PROFILE=base
# EULA (headless) acceptance for Isaac Sim containers
export OMNI_KIT_ACCEPT_EULA=YES
export ACCEPT_EULA=Y
export PRIVACY_CONSENT=Y
```

### 4. Build & push the cluster image 

##### Will pull Isaac Sim image, build Isaac Lab layers, produce/push SIF for cluster usage
##### Make sure to create an account with NGC and get an API key for NVIDIA Registry 

```
mkdir -p ~/.config/enroot
cat > ~/.config/enroot/.credentials <<'EOF'
machine nvcr.io login $oauthtoken password <YOUR_NGC_API_KEY>
EOF
chmod 600 ~/.config/enroot/.credentials
```
### 5. Set WANDB API Key

```
export WANDB_API_KEY="YOUR_WANDB_API_KEY"
```


## Optional: 

### Watch Logs 

```
squeue -u $USER
tail -f /mnt/home/$USER/isaaclab_logs/rsl_rl/Isaac-Reach-Franka-v0/**/output.log  # adjust to your Hydra dir
```
### Post-train W&B video + reward breakdown:

```
srun -p h100 --gpus=1 --container-image="$ISAACLAB_SIF" \
  --container-mounts="/mnt/home/$USER/isaaclab_logs:/mnt/home/$USER/isaaclab_logs" \
  bash -lc "
    cd /workspace/IsaacLab && \
    python cw_run/eval_and_log_wandb.py \
      --task Isaac-Reach-Franka-v0 \
      --logdir /mnt/home/$USER/isaaclab_logs/rsl_rl/Isaac-Reach-Franka-v0 \
      --ctrl_cost 1e-3 \
      --episodes 2 --video_len 300 \
      --project isaaclab-coreweave --run_name after-train
  "
  ```

