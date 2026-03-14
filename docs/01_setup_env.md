# Setup & Environment (Tested)

This page documents the **tested setup** for running LeRobot + SO100 workflows (teleop/record/train/deploy).

---

## Tested Environment (Workstation)

- OS: Ubuntu 22.04.5 LTS (Jammy)
- Kernel: 6.8.0-94-generic
- Arch: x86_64
- GPU: NVIDIA GeForce RTX 4060 (16GB)
- Driver: 575.64.03
- CUDA (driver/runtime): 12.9 (`nvidia-smi`)
- Cameras:
  - Top: /dev/video0 — 1280×720 @ 30fps (MJPG)
  - Wrist/Handeye: /dev/video4 — 1280×720 @ 30fps (MJPG)

> Optional: if you deploy on Jetson, document JetPack/L4T in a separate section.

---

## 1) Create Python environment

### conda
```bash
conda create -y -n lerobot python=3.10
conda activate lerobot
```

## 2) Install LeRobot
```bash
git clone https://github.com/huggingface/lerobot.git
cd lerobot
pip install -e .
```
Optional extras:
```bash
# simulations
pip install -e ".[aloha]"
# motor control (Feetech)
pip install -e ".[feetech]"
```

## 3) System tools (recommended)
### Camera tools
```bash
sudo apt-get update
sudo apt-get install -y v4l-utils
```
check cameras:
>Use the command to check the information of the camera and comfirm it is avalivable.
```bash
v4l2-ctl --list-devices
v4l2-ctl -d /dev/video0 --list-formats-ext
v4l2-ctl -d /dev/video4 --list-formats-ext
```
### Video decoding backend (if needed)
If you see video decode issues, install PyAV:
```bash
pip install av
```

## 4) USB serial permissions (SO100)
Quick fix (temporary):
```bash
sudo chmod 666 /dev/ttyACM0
sudo chmod 666 /dev/ttyACM1
```

## 5) HuggingFace login
```bash
huggingface-cli login
```

## 6) (AutoDL) Redirect HuggingFace cache (recommended)
To avoid running out of space on the system disk, redirect HF cache to a larger disk:
```bash
mkdir -p /root/autodl-tmp/hf_cache
rm -rf /root/.cache/huggingface
ln -s /root/autodl-tmp/hf_cache /root/.cache/huggingface
ls -ld /root/.cache/huggingface
```


