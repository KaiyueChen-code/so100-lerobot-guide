# SO100 + LeRobot Guide: Three Way Decision (SmolVLA / ACT / π0)

> A practical, reproducible guide for using the **LeRobot SO100** robotic arm in a **Three Way Decision** research project, including dataset collection, preprocessing, fine-tuning, training, evaluation, and deployment.  

## Highlights
- ✅ End-to-end workflow: **collect → format → train/finetune → evaluate → deploy**
- ✅ Covers **SmolVLA / ACT / π0** pipelines with configs + scripts
- ✅ Reproducible setup: environment, versions, and known pitfalls
- ✅ Models & datasets hosted on **HuggingFace** (GitHub keeps only code/docs)

---

## Project Overview
### Research Topic
**Three Way Decision** for SO100 robotic arm control / decision-making.

### Hardware / Stack
- Robot: **LeRobot SO100**
- Sensors: <Top camera / Wrist camera>
- Runtime: <Jetson Orin / PC + GPU / etc.>
- Framework: **LeRobot**
- Base models: **SmolVLA**, **ACT**, **π0 (pi0)**

---

## Repository Structure
```text
so100-lerobot-guide/
  README.md
  docs/                    # detailed docs
  scripts/                 # training / eval / utils
  configs/                 # model configs
  examples/                # minimal examples (optional)
  assets/                  # figures / gifs
  LICENSE
```

## Require

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
