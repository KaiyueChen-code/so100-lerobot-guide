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

Success trajectories: search → align → grasp → place

Recovery trajectories: start near failure states (misalignment / near-contact) and demonstrate correction

Example:

60× success episodes

30× recovery episodes
