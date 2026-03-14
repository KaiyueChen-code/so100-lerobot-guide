# SO100 Fine-tuning Workflow (Teleop → Record → Upload → Train → Deploy)

This doc describes the practical workflow for collecting data and fine-tuning policies on **LeRobot SO100**:
- Teleoperate (leader → follower)
- Record dataset locally
- (Optional) Upload/merge datasets on HuggingFace
- Train policies (SmolVLA / ACT / π0) on AutoDL
- Deploy & evaluate on the robot

---

## 0) Device mapping (IMPORTANT)

### Robot USB ports
In our setup:
- `/dev/ttyACM0` → **so100_follower** (robot)
- `/dev/ttyACM1` → **so100_leader** (teleop)

> Your mapping may differ. Verify with:
```bash
ls -l /dev/ttyACM*
```

### Camera device indices (IMPORTANT)
Our cameras:
top camera → /dev/video0
handeye camera → /dev/video4

Verify with:
```bash
ls -l /dev/video*
v4l2-ctl --list-devices
```
Tip: UVC cameras usually appear in pairs (0/1, 2/3, 4/5). Use the even index for the actual video stream.

## 1) USB permissions (robot serial)
```bash
sudo chmod 666 /dev/ttyACM0
sudo chmod 666 /dev/ttyACM1
```

## 2) Teleoperation (leader → follower)
With camera preview (useful for debugging)
```bash
lerobot-teleoperate \
  --robot.type=so100_follower \
  --robot.port=/dev/ttyACM1 \
  --robot.id=<FOLLOWER_ID> \
  --robot.cameras="{'handeye': {'type':'opencv', 'index_or_path':4, 'width':640, 'height':360, 'fps':30}, 'top': {'type':'opencv', 'index_or_path':0, 'width':640, 'height':360, 'fps':30}}" \
  --teleop.type=so100_leader \
  --teleop.port=/dev/ttyACM0 \
  --teleop.id=<LEADER_ID> \
  --display_data=true
```

Without preview (lighter)
```bash
lerobot-teleoperate \
  --robot.type=so100_follower \
  --robot.port=/dev/ttyACM0 \
  --robot.id=<FOLLOWER_ID> \
  --teleop.type=so100_leader \
  --teleop.port=/dev/ttyACM1 \
  --teleop.id=<LEADER_ID> \
  --display_data=false
```

## 3) Record dataset (teleop + camera)
### 3.1 Recording command template
⚠️Important: default record fps is 30fps. We strongly recommend NOT changing it, to keep the dataset consistent.

❗You should adapt num_episodes,episode_time_s,reset_times depending on your situation.

```bash
lerobot-record \
  --robot.disable_torque_on_disconnect=true \
  --robot.type=so100_follower \
  --robot.port=/dev/ttyACM0 \
  --robot.id=<FOLLOWER_ID> \
  --robot.cameras="{'handeye': {'type':'opencv', 'index_or_path':4, 'width':640, 'height':360, 'fps':30}, 'top': {'type':'opencv', 'index_or_path':0, 'width':640, 'height':360, 'fps':30}}" \
  --teleop.type=so100_leader \
  --teleop.port=/dev/ttyACM1 \
  --teleop.id=<LEADER_ID> \
  --display_data=false \
  --dataset.repo_id=<HF_DATASET_REPO_ID> \
  --dataset.num_episodes=50 \
  --dataset.episode_time_s=25 \
  --dataset.reset_time_s=10 \
  --dataset.single_task="<TASK_PROMPT>" \
  --dataset.push_to_hub=false
```

### 3.2 Continue recording on an existing dataset
add:
```bash
--resume=true
```

### 3.3 Suggested sampling plan (example)

For a grasping task, it can help to mix:

- Success trajectories: search → align → grasp → place

- Recovery trajectories: start near failure states (misalignment / near-contact) and demonstrate correction

Example:

60× success episodes

30× recovery episodes

## 4）Dataset visualization
### 4.1 Visualize episodes
```bash
lerobot-dataset-viz \
  --repo-id <HF_DATASET_REPO_ID> \
  --episode-index 0 \
  --display-compressed-images 0
```
### 4.2 Where is the local dataset stored?
LeRobot typically stores recorded datasets under:

- ~/.cache/huggingface/lerobot/<username>/<dataset_name>/

You can locate it by:
```bash
ls -lah ~/.cache/huggingface/lerobot
find ~/.cache/huggingface/lerobot -maxdepth 3 -type d | head
```

## 5) Upload dataset to HuggingFace (optional)
### 5.1 Login
```bash
huggingface-cli login
```

### 5.2 Upload local dataset folder
```bash
huggingface-cli upload <HF_USERNAME>/<DATASET_NAME> \
  ~/.cache/huggingface/lerobot/<username>/<dataset_name> \
  --repo-type dataset
```
### 5.3 Tag a dataset version(Optional)
```bash
python -c "from huggingface_hub import HfApi; HfApi().create_tag('<HF_USERNAME>/<DATASET_NAME>', tag='v3.0', repo_type='dataset')"
```
> New verison of the Lerobot needs tag='v3.0'
### 5.4 Use a mirror endpoint(Optional) 
Some networks may require:
```bash
export HF_ENDPOINT=https://hf-mirror.com
```

## 6) Merge multiple datasets
> If you want to merge different datasets into one dataset,you can follow this command
```bash
lerobot-edit-dataset \
  --repo_id <HF_USERNAME>/<MERGED_DATASET_NAME> \
  --operation.type merge \
  --operation.repo_ids "['<repo1>', '<repo2>', '<repo3>']"
```
❗ repo_ids should be <username>/<dataset_name>

## 7) AutoDL training (SmolVLA / ACT / π0)
### 7.1 Prepare environment
```bash
conda activate lerobot
```
### 7.2 Redirect HuggingFace cache (recommended)
> Establish a symbolic link to store the cache directory of HuggingFace on the data disk instead of the system disk.
1. Create a target cache directory
```bash
mkdir -p /root/autodl-tmp/hf_cache
```

2. Delete the old cache directory
```bash
rm -rf /root/.cache/huggingface
```
3. Build symbolic link
```bash
ln -s /root/autodl-tmp/hf_cache /root/.cache/huggingface
```
> Tip: You can use this command to check the symbolic link: ```ls -ld /root/.cache/huggingface```
> If you see ```/root/.cache/huggingface```->```/root/autodl-tmp/hf_cache```,that is correct.
### 7.3 Video backend note (important)
> If you encounter video decode issues while training or running your model (such as corrupted video frames or unsupported video formats), we recommend installing the `pyav` library and specifying `--dataset.video_backend=pyav` in your training or inference command.
### Steps to fix:
1. Install `pyav`:
   ```bash
   pip install av
   ```
2. Use the ```--dataset.video_backend=pyav``` argument in your command:
   ```bash
   lerobot-train \
      --policy.type=smolvla \
      --dataset.repo_id=<dataset_repo> \
      --dataset.video_backend=pyav \
      --batch_size=48 \
      --steps=20000
      ...
   ```
### 7.4 LeRobot compatibility patches (if needed)
> Only apply if you hit the specific error described in ```docs/10_troubleshooting.md.```
1. Relax torchvision version constraint：
```bash
sed -i 's/torchvision>=0.21.0/torchvision>=0.20.0/g' ~/lerobot/pyproject.toml
pip install -e . --no-deps
```
2. Fix tokenizer missing attributes:
```bash
sed -i 's/self.vlm_with_expert.processor.tokenizer.fake_image_token_id/self.vlm_with_expert.processor.tokenizer.convert_tokens_to_ids("<image>")/g' \
  ~/lerobot/src/lerobot/policies/smolvla/modeling_smolvla.py

sed -i 's/self.vlm_with_expert.processor.tokenizer.global_image_token_id/self.vlm_with_expert.processor.tokenizer.convert_tokens_to_ids("<global_image>")/g' \
  ~/lerobot/src/lerobot/policies/smolvla/modeling_smolvla.py
```

## 8) Training commands
### SmolVLA
```bash
lerobot-train \
  --policy.type=smolvla \
  --dataset.repo_id=<HF_DATASET_REPO_ID> \
  --batch_size=48 \
  --steps=40000 \
  --save_freq=5000 \
  --output_dir=/root/autodl-tmp/outputs/train/so100_smolvla \
  --job_name=so100_smolvla \
  --policy.device=cuda \
  --wandb.enable=false \
  --policy.push_to_hub=false \
  --dataset.video_backend=pyav
```

### ACT
```bash
lerobot-train \
  --policy.type=act \
  --dataset.repo_id=<HF_DATASET_REPO_ID> \
  --batch_size=8 \
  --steps=20000 \
  --save_freq=5000 \
  --output_dir=/root/autodl-tmp/outputs/train/so100_act \
  --job_name=so100_act \
  --policy.device=cuda \
  --wandb.enable=false \
  --policy.push_to_hub=false \
  --dataset.video_backend=pyav
```

### π0 (pi0)
```bash
lerobot-train \
  --policy.type=pi0 \
  --dataset.repo_id=<HF_DATASET_REPO_ID> \
  --batch_size=4 \
  --steps=40000 \
  --save_freq=5000 \
  --output_dir=/root/autodl-tmp/outputs/train/so100_pi0 \
  --job_name=so100_pi0 \
  --policy.device=cuda \
  --wandb.enable=false \
  --policy.push_to_hub=false \
  --dataset.video_backend=pyav
```
!!!!这里需要补充参数设置

### Resume from checkpoint
```bash
--resume=true \
--config_path=/root/autodl-tmp/outputs/train/<job>/checkpoints/<step>/pretrained_model/train_config.json
```

## 9) Export & upload trained model to HuggingFace
Login:
```bash
huggingface-cli login
```

Upload ```pretrained_model``` folder (contains weights + config):
```bash
huggingface-cli upload <HF_USERNAME>/<MODEL_REPO_NAME> \
  /root/autodl-tmp/outputs/train/<job>/checkpoints/<step>/pretrained_model \
  --repo-type model
```
## 10) Deployment / evaluation on robot
>Recommended before inference:
```bash
export PYTORCH_CUDA_ALLOC_CONF=expandable_segments:True
```
Run evaluation-style recording with a policy:
```bash
python src/lerobot/scripts/lerobot_record.py \
  --robot.type=so100_follower \
  --robot.port=/dev/ttyACM0 \
  --robot.cameras="{'handeye': {'type':'opencv', 'index_or_path':4, 'width':640, 'height':360, 'fps':30}, 'top': {'type':'opencv', 'index_or_path':0, 'width':640, 'height':360, 'fps':30}}" \
  --policy.path=<HF_OR_LOCAL_POLICY_PATH> \
  --dataset.repo_id=<EVAL_DATASET_REPO_ID> \
  --dataset.single_task="<TASK_PROMPT>" \
  --dataset.push_to_hub=false \
  --policy.device=cuda \
  --policy.use_amp=true \
  --display_data=false \
  --dataset.episode_time_s=20 \
  --dataset.num_episodes=20 \
  --dataset.reset_time_s=0 \
  --policy.n_action_steps=12
```
If needed:
```bash
export HF_ENDPOINT=https://hf-mirror.com
```






