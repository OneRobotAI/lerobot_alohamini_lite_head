# lerobot_alohamini_lite_head

AlohaMini robot support with **head pitch servo** (STS3215, ID 12). Built on HuggingFace LeRobot. Supports dual-arm teleoperation with mobile base + lift + head pitch (all models: 5-DoF and 6-DoF).

This variant adds a head pitch degree of freedom to all robot models, controlled via keyboard (`i`/`k`).

## Documentation

| Guide | Language | Description |
|-------|----------|-------------|
| [AlohaMini 操作手册](docs/alohamini/alohamini-lite-zh.md) | 中文 | 校准 → 遥控 → 录制 → 训练 → 评估 完整流程（含 head_pitch） |
| [AlohaMini Operation Guide](docs/alohamini/alohamini-lite-en.md) | English | Full workflow: calibration → teleoperation → recording → training → evaluation (with head_pitch) |
| [Install](docs/alohamini/install.md) | English | Environment setup, serial port permissions, HuggingFace configuration |
| [Hardware Profiles](docs/alohamini/profiles.md) | English | `--arm_profile` and `--robot_model` flag reference |

## Hardware Support

| Model | DoF | Arm Motors | Head Pitch | Base | Lift | Lead Screw |
|-------|-----|-----------|-----------|------|------|------------|
| AlohaMini 1 Head (SO-ARM) | 5-DoF | All STS3215 | STS3215 (ID 12) | STS3215 | STS3215 | 84 mm/rev |
| AlohaMini 2 Head | 6-DoF | Mixed STS3215/3095 | STS3215 (ID 12) | STS3215 | STS3095 | 131 mm/rev |
| AlohaMini 2 Pro Head | 6-DoF | All STS3250 | STS3215 (ID 12) | STS3250 | STS3095 | 131 mm/rev |
| **AlohaMini 2 Lite Head** | **6-DoF** | **All STS3215** | **STS3215 (ID 12)** | **STS3215** | **STS3215** | **84 mm/rev** |

---

## Acknowledgements

- [LeRobot](https://github.com/huggingface/lerobot) — the software stack this repository targets
- [liyiteng/lerobot_alohamini](https://github.com/liyiteng/lerobot_alohamini) — upstream AlohaMini software layer
- [ALOHA](https://tonyzhaozh.github.io/aloha/) — the bimanual teleoperation paradigm
- [SO-ARM100](https://github.com/TheRobotStudio/SO-ARM100) — pioneered the low-cost open arm design pattern
