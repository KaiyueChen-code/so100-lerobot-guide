# Hierarchical VLA Architecture for Long-Horizon Robotic Manipulation

> A practical, reproducible guide for using the **LeRobot SO100** robotic arm in a **Three Way Decision** research project, including dataset collection, preprocessing, fine-tuning, training, evaluation, and deployment.  

<p align="center">
  <img alt="LeRobot, Hugging Face Robotics Library" src="asserts/lerobot-logo-thumbnail.png" width="100%">
</p>

<div align="center">

[![Tests](https://github.com/huggingface/lerobot/actions/workflows/latest_deps_tests.yml/badge.svg?branch=main)](https://github.com/huggingface/lerobot/actions/workflows/latest_deps_tests.yml?query=branch%3Amain)
[![Tests](https://github.com/huggingface/lerobot/actions/workflows/docker_publish.yml/badge.svg?branch=main)](https://github.com/huggingface/lerobot/actions/workflows/docker_publish.yml?query=branch%3Amain)
[![Python versions](https://img.shields.io/pypi/pyversions/lerobot)](https://www.python.org/downloads/)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](https://github.com/huggingface/lerobot/blob/main/LICENSE)
[![Status](https://img.shields.io/pypi/status/lerobot)](https://pypi.org/project/lerobot/)
[![Version](https://img.shields.io/pypi/v/lerobot)](https://pypi.org/project/lerobot/)
[![Contributor Covenant](https://img.shields.io/badge/Contributor%20Covenant-v2.1-ff69b4.svg)](https://github.com/huggingface/lerobot/blob/main/CODE_OF_CONDUCT.md)
[![Discord](https://img.shields.io/badge/Discord-Join_Us-5865F2?style=flat&logo=discord&logoColor=white)](https://discord.gg/q8Dzzpym3f)

</div>

**LeRobot** aims to provide models, datasets, and tools for real-world robotics in PyTorch. The goal is to lower the barrier to entry so that everyone can contribute to and benefit from shared datasets and pretrained models.

🤗 A hardware-agnostic, Python-native interface that standardizes control across diverse platforms, from low-cost arms (SO-100) to humanoids.

🤗 A standardized, scalable LeRobotDataset format (Parquet + MP4 or images) hosted on the Hugging Face Hub, enabling efficient storage, streaming and visualization of massive robotic datasets.

🤗 State-of-the-art policies that have been shown to transfer to the real-world ready for training and deployment.

🤗 Comprehensive support for the open-source ecosystem to democratize physical AI.

## Highlights
- ✅ End-to-end workflow: **collect → format → train/finetune → evaluate → deploy**
- ✅ Covers **SmolVLA / ACT / π0** pipelines with configs + scripts
- ✅ Reproducible setup: environment, versions, and known pitfalls
- ✅ Models & datasets hosted on **HuggingFace** (GitHub keeps only code/docs)

---

## Overview
This project proposes a hierarchical Vision-Language-Action (VLA) framework for robotic arms, designed to tackle the core challenges of long-horizon manipulation tasks: error accumulation, tight coupling between planning and execution, high inference latency, and insufficient safety supervision.
Inspired by the structure of the human nervous system, the architecture is organized into three decoupled modules:

- 🧠 Brain — High-level semantic understanding and task planning. A fine-tuned Qwen large language model (via LoRA) receives natural language instructions and real-time visual feedback, decomposing complex goals into ordered sub-task sequences.
- 🦾 Cerebellum — Low-level action execution. Dedicated VLA/action models (ACT, SmolVLA, Pi0) translate sub-task commands into precise robotic arm motions, handling grasping, transport, and placement.
- 👁️ Supervisor — Real-time safety monitoring and recovery. A YOLO-based detection module watches for hazards (e.g., human hands entering the workspace) and triggers a pause → observe → confirm → resume loop rather than an abrupt stop, enabling safe and continuous operation in human-robot collaboration settings.

More overview details：
- [Setup & Environment](docs/00_overview.md)

### Key Goals
- Improve task success rate on multi-step, long-horizon manipulation tasks
- Reduce end-to-end inference latency through local deployment of LLM + VLA models
- Enable robust, recoverable safety supervision in open environments
- Explore diverse data collection methods (teleoperation, VR control, UMI) to improve model generalization

### Current Status
A working demo system has been built and validated on the SO100 robotic arm platform, with ACT, SmolVLA, and Pi0 already fine-tuned and tested. The project is actively progressing toward full three-module integration, hardware upgrade, and localized inference deployment.



### Hardware / Stack
- Robot: **LeRobot SO100**
- Sensors: <Top camera / Wrist camera>
- Runtime: PC workstation (x86_64) + NVIDIA GeForce RTX 4060 (16GB)
- Framework: **LeRobot**
- Base models: **SmolVLA**, **ACT**, **π0 (pi0)**

---

## Repository Structure
```text
so100-lerobot-guide/
  README.md
  docs/                    # detailed docs
  configs/                 # model configs
  Assets/                  # figures / gifs
  LICENSE
```

## Requirements
- OS: Ubuntu 22.04.5 LTS (Jammy)
- Kernel: 6.8.0-94-generic
- CPU Arch: x86_64
- GPU: NVIDIA GeForce RTX 4060 (16GB)
- NVIDIA Driver: 575.64.03
- CUDA (driver/runtime): 12.9 (`nvidia-smi`)
- Cameras (UVC):
  - Top: DSJ-2062-309 — **1280×720 @ 30fps (MJPG)**
  - Wrist: DSJ-2062-309 — **1280×720 @ 30fps (MJPG)**

## Quick Start
If you just want to run inference / reproduce training quickly, follow this.

### 1) Setup environment
```bash
#create a virtual environment with Python 3.10, using conda:
conda create -y -n lerobot python=3.10
```

### 2) Install LeRobot
```bash
#step1 clone the repository and navigate into the directory:
git clone https://github.com/huggingface/lerobot.git
cd lerobot

#step2 install the library in editable mode:
pip install -e .

#Optional dependencies
#Simulations:
pip install -e ".[aloha]"
#Motor Control:
pip install -e ".[feetech]"
```

### 3) HuggingFace login
```bash
huggingface-cli login
```

### 4) Download models/datasets
Large artifacts (datasets / checkpoints) are hosted on HuggingFace:

- Dataset(s):
  - EricChen06/so100_smolvla
  - <optional_more_datasets>

- Model checkpoints:
  - SmolVLA: EricChen06/so100_smolvla
  - ACT: EricChen06/so100_act
  - π0: EricChen06/so100_pi0
  
🔗Link: https://huggingface.co/EricChen06

### 5) Teleop quick test (SO100 leader → follower)
```bash
lerobot-teleoperate \
  --robot.type=so100_follower \
  --robot.port=/dev/ttyACM0 \
  --robot.id=<FOLLOWER_ID> \
  --robot.cameras="{'handeye': {'type':'opencv', 'index_or_path':4, 'width':640, 'height':360, 'fps':30}, 'top': {'type':'opencv', 'index_or_path':0, 'width':640, 'height':360, 'fps':30}}" \
  --teleop.type=so100_leader \
  --teleop.port=/dev/ttyACM1 \
  --teleop.id=<LEADER_ID> \
  --display_data=true
```
If you see permission errors:
```bash
sudo usermod -aG dialout $USER
newgrp dialout
# or temporary:
sudo chmod 666 /dev/ttyACM0 /dev/ttyACM1
```


### 6）Record a tiny dataset (1 episode)
```bash
lerobot-record \
  --robot.type=so100_follower \
  --robot.port=/dev/ttyACM0 \
  --robot.id=<FOLLOWER_ID> \
  --robot.cameras="{'handeye': {'type':'opencv', 'index_or_path':4, 'width':640, 'height':360, 'fps':30}, 'top': {'type':'opencv', 'index_or_path':0, 'width':640, 'height':360, 'fps':30}}" \
  --teleop.type=so100_leader \
  --teleop.port=/dev/ttyACM1 \
  --teleop.id=<LEADER_ID> \
  --display_data=false \
  --dataset.repo_id=<HF_DATASET_REPO_ID> \
  --dataset.num_episodes=1 \
  --dataset.episode_time_s=10 \
  --dataset.reset_time_s=3 \
  --dataset.single_task="Grasp the blue cube" \
  --dataset.push_to_hub=false
```
Visualize:
```bash
lerobot-dataset-viz --repo-id <HF_DATASET_REPO_ID> --episode-index 0
```

### Next steps
Full SO100 workflow (teleop/record/upload/train/deploy):
- [Setup & Environment](docs/01_setup_env.md)

Setup & environment details:
- [SO100 Fine-tuning Workflow](docs/02_so100_finetune_workflow.md)


### Acknowledgement

This project builds upon following open-source code-bases. Please visit the URLs to see the respective LICENSES (If you find these projects valuable, it would be greatly appreciated if you could give them a star rating.):

1. https://github.com/huggingface/lerobot



