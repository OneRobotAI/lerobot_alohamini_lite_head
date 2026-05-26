# AlohaMini 完整操作流程（中文版）

> 前置条件：先完成 [install.md](install.md) 的安装。
> 硬件配置说明见 [profiles.md](profiles.md)。

双机配置 — **PC（客户端）** + **树莓派（服务端）** 在同一局域网内。

---

## 适配代码

### 第一步：打开 `src/lerobot/robots/alohamini/lekiwi.py`

#### 1.1 添加 arm profile

找到 `_ARM_PROFILES` 字典（大概 48 行），看到 `"am-follower-6dof"` 这一段，在它**之后**加上一段新的 `"am-follower-6dof-lite"`：

```python
    "am-follower-6dof": (
        ("shoulder_pan", 1, "sts3215", None),
        ("shoulder_lift", 2, "sts3095", None),
        ("elbow_flex", 3, "sts3095", None),
        ("wrist_flex", 4, "sts3215", None),
        ("wrist_yaw", 5, "sts3215", None),
        ("wrist_roll", 6, "sts3215", None),
        ("gripper", 7, "sts3215", MotorNormMode.RANGE_0_100),
    ),

    # 👇 在这里插入新 profile
    "am-follower-6dof-lite": (
        ("shoulder_pan", 1, "sts3215", None),
        ("shoulder_lift", 2, "sts3215", None),
        ("elbow_flex", 3, "sts3215", None),
        ("wrist_flex", 4, "sts3215", None),
        ("wrist_yaw", 5, "sts3215", None),
        ("wrist_roll", 6, "sts3215", None),
        ("gripper", 7, "sts3215", MotorNormMode.RANGE_0_100),
    ),
```

和原来的区别：`shoulder_lift` 和 `elbow_flex` 从 `sts3095` **改为 `sts3215`**。

#### 1.2 添加 robot spec

找到 `_ROBOT_SPECS` 字典（大概 112 行），看到 `"alohamini2pro-head"` 这一段，在它**之后**（`}` 后面加逗号，然后插入）：
```python
    "alohamini2pro-head": {
        "arm_profile": "am-follower-6dof-hd",
        "base_motor": "sts3250",
        "lift_motor": "sts3095",
        "lead_mm_per_rev": 131.0,
    },

    # 👇 在这里插入新 spec
    "alohamini2lite-head": {
        "arm_profile": "am-follower-6dof-lite",
        "base_motor": "sts3215",
        "lift_motor": "sts3215",
        "lead_mm_per_rev": 84.0,
    },
```

---

### 第二步：打开 `src/lerobot/robots/alohamini/config_lekiwi.py`（可选）

找到 `robot_model: str = "alohamini1"` 上面的注释，在 `alohamini2pro-head` 那行后面加一行：

```
    # alohamini2lite-head– am-follower-6dof-lite (全 STS3215 6 轴), base sts3215, lift sts3215, lead=131 mm/rev
```

以及 `LeKiwiClientConfig` 里也有类似的注释，同样补一行。

---

### 第三步：打开 `src/lerobot/robots/alohamini/lekiwi_client.py` 第 85 行

原来：

```python
_6DOF_MODELS = {"alohamini2-head", "alohamini2pro-head"}
```

改成：

```python
_6DOF_MODELS = {"alohamini2-head", "alohamini2pro-head", "alohamini2lite-head"}
```

---

### 第四步：打开 `src/lerobot/robots/alohamini/lekiwi_host.py` 第 58 行

原来：

```python
choices=["alohamini1-head", "alohamini2-head", "alohamini2pro-head"],
```

**改成：**

```python
choices=["alohamini1-head", "alohamini2-head", "alohamini2pro-head", "alohamini2lite-head"],
```

再把第 61-63 行的 help 注释后面加一行，注意要在上一行末尾先加 `\n`：

```python
help=(
    "Robot model — drives follower arm profile, base motors, lift motor, and lead screw pitch.\n"
    "  alohamini1-head   : so-arm-5dof,         base sts3215, lift sts3215, lead=84 mm/rev\n"
    "  alohamini2-head   : am-follower-6dof,    base sts3215, lift sts3095, lead=131 mm/rev\n"
    "  alohamini2pro-head: am-follower-6dof-hd, base sts3250, lift sts3095, lead=131 mm/rev\n"
    "  alohamini2lite-head: am-follower-6dof-lite, base sts3215, lift sts3215, lead=84 mm/rev"
),
```

---

### 第五步：打开 `examples/alohamini/evaluate_bi.py` 第 34-36 行

原来：

```python
parser.add_argument("--robot_model", type=str, default="alohamini1-head",
                    choices=["alohamini1-head", "alohamini2-head", "alohamini2pro-head"],
                    help="Must match the robot_model on the Pi host side")
```

**改成：**

```python
parser.add_argument("--robot_model", type=str, default="alohamini1-head",
                    choices=["alohamini1-head", "alohamini2-head", "alohamini2pro-head", "alohamini2lite-head"],
                    help="Must match the robot_model on the Pi host side")
```

### 第六步：头部俯仰舵机（head_pitch）

本版本在右总线（right_port）上新增了一个头部俯仰舵机 `head_pitch`（STS3215，ID 12），所有型号（5-DoF / 6-DoF）均支持。

代码改动已在仓库中完成，无需手动修改。

---

**PC 端需要同步的文件：**
- `record_bi.py` ✅
- `evaluate_bi.py` ✅
- `lekiwi_client.py` ✅

**树莓派端需要同步的文件：**
- `lekiwi.py` ✅
- `lekiwi_host.py` ✅

---

## 1. 系统架构

```
┌──────────────────────────────┐      局域网      ┌──────────────────────────────────┐
│          PC（客户端）         │ ◄───────────────► │         树莓派（服务端）         │
│                              │                   │                                  │
│  • 主手臂（Leader，USB）     │                   │  • 从手臂（Follower，USB）        │
│  • teleoperate_bi.py         │                   │  • 底盘轮子 + 升降轴（USB）       │
│  • record_bi.py              │                   │  • 头部俯仰舵机（USB）            │
│  • 训练 / 评估               │                   │  • 摄像头（USB）                  │
│                              │                   │  • lekiwi_host.py                │
└──────────────────────────────┘                   └──────────────────────────────────┘
```

头部俯仰舵机（`head_pitch`）连接在树莓派的 right_port 总线上（与右臂共用），ID 为 12，型号 STS3215。

两台机器必须在同一局域网，并已安装完整环境。

---

## 2. 端口配置

逐个插入设备，然后运行：

```bash
lerobot-find-port
# 或直接查看：
ls /dev/ttyACM*
```

**从手臂（Follower）** — 在树莓派上编辑 `src/lerobot/robots/alohamini/config_lekiwi.py`：

```python
@dataclass
class LeKiwiConfig(RobotConfig):
    left_port:  str = "/dev/ttyACM0"   # 替换为你的左总线端口
    right_port: str = "/dev/ttyACM1"   # 替换为你的右总线端口
```

> 头部俯仰舵机接在右总线上，无需额外端口。

**主手臂（Leader）** — 在 PC 上编辑 `examples/alohamini/teleoperate_bi.py`：

```python
left_arm_config  = SOLeaderConfig(port="/dev/ttyACM2", ...)   # 替换
right_arm_config = SOLeaderConfig(port="/dev/ttyACM3", ...)   # 替换
```

> 端口号在重新连接或重启后可能变化。如果购买的是完整 AlohaMini 整机，树莓派的从手臂端口已通过 udev 规则固定，无需额外操作。

---

## 3. 摄像头配置

```bash
lerobot-find-cameras
```

将检测到的索引填入 `src/lerobot/robots/alohamini/config_lekiwi.py`。

以下例子增加了左臂和右臂的摄像头，根据查询结果修改相应摄像头 index。

> 每个摄像头需要独立的 USB 接口——不要在同一个 USB HUB 上连接多个摄像头。
> 如果你用 Jetson/树莓派作为 Host，电脑作为 Client，需要在 Jetson 和电脑端同时修改上面摄像头参数，这样才能在后续的 rerun 中看到画面。
> 摄像头数量不宜太多，Jetson/树莓派带宽有限，建议 2 到 3 个。

---

## 4. 校准

### 步骤 1 — 校准从手臂（树莓派端）

SSH 登录树莓派，根据你的型号运行对应的 host 脚本。首次运行会提示校准：将每个关节转到机械中点 → 回车 → 向左转 90° → 回车 → 向右转 90° → 回车。

对于带头部俯仰的版本，校准时还会提示将头部俯仰关节也转到中点并记录其运动范围。

```bash
# AlohaMini 1（SO-ARM 5-DoF）
python -m lerobot.robots.alohamini.lekiwi_host --robot_model alohamini1

# AlohaMini 2（AM-ARM 6-DoF，混合 STS3215/STS3095）
python -m lerobot.robots.alohamini.lekiwi_host --robot_model alohamini2

# AlohaMini 2 Pro（AM-ARM 6-DoF，STS3250）
python -m lerobot.robots.alohamini.lekiwi_host --robot_model alohamini2pro-head

# AlohaMini 2 Lite（AM-ARM 6-DoF，全 STS3215）
python -m lerobot.robots.alohamini.lekiwi_host --robot_model alohamini2lite-head
```

> SO-ARM 5-DoF 参考中点位置：参见原版手册图示。

校准机械臂后，程序会自动校准升降台，升降台会自动下降，碰到底部时会自动停止（升降柱要和底盘连接稳固）。程序连续检测堵转（`Stalled at current...`）。随后自动断电（`Disable torque output`）并设置零点（`set-zero ... height_now=0.00 mm`）。最后打印了 `Lift axis homed to 0mm.`，这是校准成功的标志。

> 如果缺少 `lerobot[deepdiff-dep]` 依赖，安装：`pip install 'lerobot[deepdiff-dep]'`

### 步骤 2 — 校准主手臂（PC 端）

将 `<Pi_IP>` 替换为树莓派的 IP 地址。

校准主臂时，需要先开启 `lekiwi_host`，参照上一步从臂校准，例如：

```bash
python -m lerobot.robots.alohamini.lekiwi_host --robot_model alohamini2lite-head
```

**SO-ARM 主手（5-DoF）：**
```bash
python examples/alohamini/teleoperate_bi.py \
  --remote_ip <Pi_IP> \
  --leader_id so101_leader_bi \
  --arm_profile so-arm-5dof
```

**AM-ARM 主手（6-DoF）：**
```bash
python examples/alohamini/teleoperate_bi.py \
  --remote_ip <Pi_IP> \
  --leader_id am_leader_bi \
  --arm_profile am-leader-6dof
```

> 校准完成后，重新上电主手和从手臂，使校准生效。
> 如果缺少 `rerun-sdk` 依赖，安装：`pip install 'lerobot[viz]'`

---

## 5. 遥控操作

先启动树莓派端 host，再启动 PC 端 client（校准已完成，跳过）：

**树莓派 — 按型号启动 host：**

```bash
# AlohaMini 1
python -m lerobot.robots.alohamini.lekiwi_host --robot_model alohamini1-head

# AlohaMini 2
python -m lerobot.robots.alohamini.lekiwi_host --robot_model alohamini2-head

# AlohaMini 2 Pro
python -m lerobot.robots.alohamini.lekiwi_host --robot_model alohamini2pro-head

# AlohaMini 2 Lite
python -m lerobot.robots.alohamini.lekiwi_host --robot_model alohamini2lite-head
```

**PC — 按主手型号启动 client：**

```bash
# SO-ARM 主手（5-DoF）
python examples/alohamini/teleoperate_bi.py \
  --remote_ip <Pi_IP> --leader_id so101_leader_bi --arm_profile so-arm-5dof

# AM-ARM 主手（6-DoF）
python examples/alohamini/teleoperate_bi.py \
  --remote_ip <Pi_IP> --leader_id am_leader_bi --arm_profile am-leader-6dof
```

**操作说明：**
- 手动转主手臂的关节，看从臂是否跟着动
- 键盘控制底盘：`w/s` 前后，`z/x` 左右平移，`a/d` 旋转，`u/j` 升降
- **头部俯仰：`i` 抬头，`k` 低头**
- 按 `q` 退出

> 头部俯仰使用归一化位置控制（-1.0 到 1.0），每次按键步进 0.05。
> 如果出现相机 FPS 不够用，可以修改 fps。有时候由于带宽或环境因素，相机会自动降低 fps 到 25。

---

## 6. 录制数据集

> 录制前请确保树莓派端 host 已在运行（见第 5 节）。
> `--arm_profile` 指的是主手臂硬件，而非从手机器人。
> 将 `<Pi_IP>` 替换为树莓派的 IP 地址。

收集前请在 `examples/alohamini/record_bi.py` 脚本中修改主臂 port 端口号。

### AlohaMini 1 — SO-ARM 主手（5-DoF）— PC

**创建新数据集：**
```bash
python examples/alohamini/record_bi.py \
  --dataset $HF_USER/so100_bi_test \
  --num_episodes 1 \
  --fps 30 \
  --episode_time 45 \
  --reset_time 8 \
  --task_description "pickup1" \
  --remote_ip <Pi_IP> \
  --leader_id so101_leader_bi \
  --arm_profile so-arm-5dof
```

**续录已有数据集（加 `--resume`）：**
```bash
python examples/alohamini/record_bi.py \
  --dataset $HF_USER/so100_bi_test \
  --num_episodes 1 \
  --fps 30 \
  --episode_time 45 \
  --reset_time 8 \
  --task_description "pickup1" \
  --remote_ip <Pi_IP> \
  --leader_id so101_leader_bi \
  --arm_profile so-arm-5dof \
  --resume
```

### AlohaMini 2 / 2 Pro / 2 Lite — AM-ARM 主手（6-DoF）— PC

先修改 `examples/alohamini/record_bi.py`：

在 `--arm_profile` 和 `--resume` 之间插入：

```python
parser.add_argument("--robot_model", type=str, default="alohamini1-head",
                    choices=["alohamini1-head", "alohamini2-head", "alohamini2pro-head", "alohamini2lite-head"],
                    help="Must match the robot_model on the Pi host side")
```

将 `LeKiwiClientConfig` 行修改为：

```python
robot_config = LeKiwiClientConfig(remote_ip=args.remote_ip, id=args.robot_id,
                              robot_model=args.robot_model)
```

**创建新数据集：**
```bash
python examples/alohamini/record_bi.py \
  --dataset $HF_USER/am2_bi_test \
  --num_episodes 1 \
  --fps 30 \
  --episode_time 45 \
  --reset_time 8 \
  --task_description "pickup1" \
  --remote_ip 192.168.1.9 \
  --leader_id am_leader_bi \
  --arm_profile am-leader-6dof \
  --robot_model alohamini2lite-head
```

**续录已有数据集（加 `--resume`）：**
```bash
python examples/alohamini/record_bi.py \
  --dataset $HF_USER/am2_bi_test \
  --num_episodes 1 \
  --fps 30 \
  --episode_time 45 \
  --reset_time 8 \
  --task_description "pickup1" \
  --remote_ip <Pi_IP> \
  --leader_id am_leader_bi \
  --arm_profile am-leader-6dof \
  --robot_model alohamini2lite \
  --resume
```

> 录制时头部俯仰动作通过键盘 `i`/`k` 控制，会被自动记录到数据集中。
> 如果缺少 `datasets` 依赖，安装：`pip install 'lerobot[dataset]'`
> 如果缺少 `socksio` 依赖，安装：`pip install 'httpx[socks]'`

---

## 7. 数据集回放（可选）

```bash
python examples/alohamini/replay_bi.py \
  --dataset $HF_USER/am2_bi_test \
  --episode 0 \
  --remote_ip <Pi_IP>
```

---

## 8. 数据集可视化（可选）

```bash
lerobot-dataset-viz \
  --repo-id $HF_USER/am2_bi_test \
  --episode-index 0 \
  --display-compressed-images
```

---

## 9. 训练

### 本地训练

```bash
lerobot-train \
  --dataset.repo_id=$HF_USER/am2_bi_test \
  --policy.type=act \
  --output_dir=outputs/train/act_your_dataset1 \
  --job_name=act_your_dataset \
  --policy.device=cuda \
  --wandb.enable=false \
  --policy.repo_id=$HF_USER/act_policy \
  --dataset.video_backend=pyav
```

`steps` 默认 100000，调整加参数 `--steps`。

> 如果缺少 `accelerate` 依赖，安装：`pip install 'lerobot[training]'`

### 没有本地 GPU？

使用任意云 GPU 服务商（如 Featurize、AutoDL、Lambda Labs、Vast.ai）。按同样方式搭建环境、运行相同训练命令，然后将 checkpoint 复制回本地进行评估。

---

## 10. 评估

确保树莓派端 host 已在运行（第 5 节），然后从 PC 端运行推理。

> `--robot_model` / `--robot.robot_model` **必须与树莓派端 host 运行的型号一致：**
> - `alohamini1` — SO-ARM 5-DoF，**17 维状态**（含 head_pitch）
> - `alohamini2` / `alohamini2pro` / `alohamini2lite` — AM-ARM 6-DoF，**19 维状态**（含 head_pitch）

### 方式 A — evaluate_bi.py（自定义脚本，N 个 episode，自动上传至 Hub）

```bash
python examples/alohamini/evaluate_bi.py \
  --num_episodes 3 \
  --fps 20 \
  --episode_time 45 \
  --task_description "Pick and place task" \
  --hf_model_id outputs/train/act_your_dataset1/checkpoints/020000/pretrained_model \
  --hf_dataset_id $HF_USER/eval_act_policy \
  --remote_ip <Pi_IP> \
  --robot_id my_alohamini \
  --robot_model alohamini2lite
```

其他型号同理，将 `--robot_model` 替换为：`alohamini1`、`alohamini2`、`alohamini2pro`。

### 方式 B — lerobot-rollout（官方 CLI）— 暂时不可用

纯推理，不录制：

```bash
python -m lerobot.scripts.lerobot_rollout \
  --strategy.type=base \
  --robot.type=alohamini_client \
  --robot.remote_ip=<Pi_IP> \
  --robot.robot_model=alohamini2lite \
  --policy.path=outputs/train/act_your_dataset1/checkpoints/020000/pretrained_model \
  --task="Pick and place task" \
  --fps=20 \
  --duration=45 \
  --display_data=true
```

推理 + 录制评估数据集（数据集名称必须以 `rollout_` 开头）：

```bash
python -m lerobot.scripts.lerobot_rollout \
  --strategy.type=sentry \
  --robot.type=alohamini_client \
  --robot.remote_ip=<Pi_IP> \
  --robot.robot_model=alohamini2lite \
  --policy.path=outputs/train/act_your_dataset1/checkpoints/020000/pretrained_model \
  --dataset.repo_id=$HF_USER/rollout_eval1 \
  --task="Pick and place task" \
  --fps=20 \
  --display_data=true
```

其他型号同理，将 `--robot.robot_model` 替换为对应值。

---

## 附录：硬件配置对照表

| `--robot_model` | 从手臂型号 | 关节数/臂 | 臂电机 | 头部俯仰 | 底盘电机 | 升降电机 | 丝杠导程 |
|---|---|---|---|---|---|---|---|
| `alohamini1` | so-arm-5dof | 6（5 关节 + 夹爪） | 全 STS3215 | STS3215 (ID 12) | STS3215 | STS3215 | 84 mm/rev |
| `alohamini2` | am-follower-6dof | 7（6 关节 + 夹爪） | 混合 STS3215/STS3095 | STS3215 (ID 12) | STS3215 | STS3095 | 131 mm/rev |
| `alohamini2pro` | am-follower-6dof-hd | 7（6 关节 + 夹爪） | 全 STS3250 | STS3215 (ID 12) | STS3250 | STS3095 | 131 mm/rev |
| `alohamini2lite` | am-follower-6dof-lite | 7（6 关节 + 夹爪） | 全 STS3215 | STS3215 (ID 12) | STS3215 | STS3215 | 84 mm/rev |
