# SO100 + LeRobot Guide: Tri-Attribute Decision (SmolVLA / ACT / π0)

> A practical, reproducible guide for using the **LeRobot SO100** robotic arm in a **Three Way Decision** research project, including dataset collection, preprocessing, fine-tuning, training, evaluation, and deployment.  

## Highlights
- ✅ End-to-end workflow: **collect → format → train/finetune → evaluate → deploy**
- ✅ Covers **SmolVLA / ACT / π0** pipelines with configs + scripts
- ✅ Reproducible setup: environment, versions, and known pitfalls
- ✅ Models & datasets hosted on **HuggingFace** (GitHub keeps only code/docs)

---

## Project Overview
### Research Topic
**Tri-Attribute Decision** for SO100 robotic arm control / decision-making.

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
