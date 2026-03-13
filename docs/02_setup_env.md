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

