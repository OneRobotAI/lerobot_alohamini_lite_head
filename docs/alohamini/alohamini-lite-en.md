# AlohaMini — Complete Operation Guide (English)

> Prerequisites: complete [install.md](install.md) first.
> Hardware configuration reference: [profiles.md](profiles.md).

Two-machine setup — **PC (client)** + **Raspberry Pi (server)** on the same LAN.

---

## Code Adaptation

### Step 1: Edit `src/lerobot/robots/alohamini/lekiwi.py`

#### 1.1 Add arm profile

Find the `_ARM_PROFILES` dict (around line 48). Locate `"am-follower-6dof"` and insert a new `"am-follower-6dof-lite"` entry **after** it:

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

    # Insert new profile here
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

Difference: `shoulder_lift` and `elbow_flex` changed from `sts3095` to `sts3215`.

#### 1.2 Add robot spec

Find the `_ROBOT_SPECS` dict (around line 112). Locate `"alohamini2pro-head"` and insert a new `"alohamini2lite-head"` entry **after** it (add a comma after the closing `}`, then insert):
```python
    "alohamini2pro-head": {
        "arm_profile": "am-follower-6dof-hd",
        "base_motor": "sts3250",
        "lift_motor": "sts3095",
        "lead_mm_per_rev": 131.0,
    },

    # Insert new spec here
    "alohamini2lite-head": {
        "arm_profile": "am-follower-6dof-lite",
        "base_motor": "sts3215",
        "lift_motor": "sts3215",
        "lead_mm_per_rev": 84.0,
    },
```

---

### Step 2: Edit `src/lerobot/robots/alohamini/config_lekiwi.py` (optional)

Find the comment above `robot_model: str = "alohamini1"`. Append a line after the `alohamini2pro-head` line:

```
    # alohamini2lite-head– am-follower-6dof-lite (all STS3215 6-DoF), base sts3215, lift sts3215, lead=84 mm/rev
```

Similarly, add the same line to the comment block inside `LeKiwiClientConfig`.

---

### Step 3: Edit `src/lerobot/robots/alohamini/lekiwi_client.py` line 85

**Before:**

```python
_6DOF_MODELS = {"alohamini2-head", "alohamini2pro-head"}
```

**After:**

```python
_6DOF_MODELS = {"alohamini2-head", "alohamini2pro-head", "alohamini2lite-head"}
```

---

### Step 4: Edit `src/lerobot/robots/alohamini/lekiwi_host.py` line 58

**Before:**

```python
choices=["alohamini1-head", "alohamini2-head", "alohamini2pro-head"],
```

**After:**

```python
choices=["alohamini1-head", "alohamini2-head", "alohamini2pro-head", "alohamini2lite-head"],
```

Update the help text (append a line, ensure previous line ends with `\n`):

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

### Step 5: Edit `examples/alohamini/evaluate_bi.py` lines 34-36

**Before:**

```python
parser.add_argument("--robot_model", type=str, default="alohamini1-head",
                    choices=["alohamini1-head", "alohamini2-head", "alohamini2pro-head"],
                    help="Must match the robot_model on the Pi host side")
```

**After:**

```python
parser.add_argument("--robot_model", type=str, default="alohamini1-head",
                    choices=["alohamini1-head", "alohamini2-head", "alohamini2pro-head", "alohamini2lite-head"],
                    help="Must match the robot_model on the Pi host side")
```

### Step 6: Head Pitch Servo (head_pitch)

This version adds a head pitch servo `head_pitch` (STS3215, ID 12) on the right bus (`right_port`). All robot models (5-DoF / 6-DoF) include this extra degree of freedom.

The code changes are already applied in this repository — no manual edits needed.

---

**Files to sync to PC:**
- `record_bi.py` ✅
- `evaluate_bi.py` ✅
- `lekiwi_client.py` ✅

**Files to sync to Raspberry Pi:**
- `lekiwi.py` ✅
- `lekiwi_host.py` ✅

---

## 1. System Architecture

```
┌──────────────────────────┐       LAN        ┌────────────────────────────────┐
│       PC (Client)        │ ◄───────────────► │    Raspberry Pi (Server)       │
│                          │                   │                                │
│  • Leader arm (USB)      │                   │  • Follower arm (USB)          │
│  • teleoperate_bi.py     │                   │  • Base wheels + lift (USB)    │
│  • record_bi.py          │                   │  • Head pitch servo (USB)      │
│  • Training / Evaluation │                   │  • Cameras (USB)               │
│                          │                   │  • lekiwi_host.py              │
└──────────────────────────┘                   └────────────────────────────────┘
```

The head pitch servo (`head_pitch`) connects to the Pi's `right_port` bus (shared with the right arm), ID 12, STS3215.

Both machines must be on the same LAN with the full environment installed.

---

## 2. Port Configuration

Plug in devices one at a time, then run:

```bash
lerobot-find-port
# Or check directly:
ls /dev/ttyACM*
```

**Follower arm** — Edit `src/lerobot/robots/alohamini/config_lekiwi.py` on the Pi:

```python
@dataclass
class LeKiwiConfig(RobotConfig):
    left_port:  str = "/dev/ttyACM0"   # Replace with your left bus port
    right_port: str = "/dev/ttyACM1"   # Replace with your right bus port
```

> The head pitch servo is on the right bus — no extra port needed.

**Leader arm** — Edit `examples/alohamini/teleoperate_bi.py` on the PC:

```python
left_arm_config  = SOLeaderConfig(port="/dev/ttyACM2", ...)   # Replace
right_arm_config = SOLeaderConfig(port="/dev/ttyACM3", ...)   # Replace
```

> Ports may change after reconnection or reboot. If you bought a complete AlohaMini system, the Pi's follower arm ports are fixed via udev rules — no extra action needed.

---

## 3. Camera Configuration

```bash
lerobot-find-cameras
```

Fill the detected indices into `src/lerobot/robots/alohamini/config_lekiwi.py`.

The example below adds a left and right arm camera — adjust the camera indices based on your query results.

> Each camera needs a dedicated USB interface — do not connect multiple cameras on the same USB hub.
> If you use Jetson/Raspberry Pi as Host and PC as Client, you need to modify the camera parameters on both the Jetson and PC sides for rerun visualization to work.
> Don't use too many cameras — Jetson/Raspberry Pi have limited USB bandwidth. 2–3 cameras recommended.

---

## 4. Calibration

### Step 1 — Calibrate Follower Arm (Raspberry Pi)

SSH into the Pi. Run the corresponding host script for your model. On first run, it will prompt: move each joint to mechanical center → Enter → rotate left 90° → Enter → rotate right 90° → Enter.

For the head pitch version, the calibration also includes the head pitch joint — move it to center, then through its full range of motion.

```bash
# AlohaMini 1 (SO-ARM 5-DoF)
python -m lerobot.robots.alohamini.lekiwi_host --robot_model alohamini1-head

# AlohaMini 2 (AM-ARM 6-DoF, mixed STS3215/STS3095)
python -m lerobot.robots.alohamini.lekiwi_host --robot_model alohamini2-head

# AlohaMini 2 Pro (AM-ARM 6-DoF, STS3250)
python -m lerobot.robots.alohamini.lekiwi_host --robot_model alohamini2pro-head

# AlohaMini 2 Lite (AM-ARM 6-DoF, all STS3215)
python -m lerobot.robots.alohamini.lekiwi_host --robot_model alohamini2lite-head
```

> SO-ARM 5-DoF center position reference: see the original manual diagrams.

After arm calibration, the lift axis auto-calibrates: it descends until it hits the bottom (ensure lift column is firmly connected to base). The program detects stall (`Stalled at current...`), then disables torque (`Disable torque output`) and sets zero (`set-zero ... height_now=0.00 mm`). `Lift axis homed to 0mm.` signals success.

> If `lerobot[deepdiff-dep]` is missing: `pip install 'lerobot[deepdiff-dep]'`

### Step 2 — Calibrate Leader Arm (PC)

Replace `<Pi_IP>` with the Pi's IP address.

The host must already be running when calibrating the leader (refer to Step 1). For example:

```bash
python -m lerobot.robots.alohamini.lekiwi_host --robot_model alohamini2lite-head
```

**SO-ARM leader (5-DoF):**
```bash
python examples/alohamini/teleoperate_bi.py \
  --remote_ip <Pi_IP> \
  --leader_id so101_leader_bi \
  --arm_profile so-arm-5dof
```

**AM-ARM leader (6-DoF):**
```bash
python examples/alohamini/teleoperate_bi.py \
  --remote_ip <Pi_IP> \
  --leader_id am_leader_bi \
  --arm_profile am-leader-6dof
```

> After calibration, power-cycle both the leader and follower arms for calibration to take effect.
> If `rerun-sdk` is missing: `pip install 'lerobot[viz]'`

---

## 5. Teleoperation

Start the Pi host first, then the PC client (calibration is already done, skip it):

**Pi — start host by model:**

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

**PC — start client by leader type:**

```bash
# SO-ARM leader (5-DoF)
python examples/alohamini/teleoperate_bi.py \
  --remote_ip <Pi_IP> --leader_id so101_leader_bi --arm_profile so-arm-5dof

# AM-ARM leader (6-DoF)
python examples/alohamini/teleoperate_bi.py \
  --remote_ip <Pi_IP> --leader_id am_leader_bi --arm_profile am-leader-6dof
```

**Controls:**
- Manually move the leader arm joints — the follower should mirror them
- Keyboard base control: `w/s` forward/back, `z/x` strafe left/right, `a/d` rotate, `u/j` lift up/down
- **Head pitch: `i` tilt up, `k` tilt down**
- Press `q` to quit

> Head pitch uses normalized position control (-1.0 to 1.0), stepping 0.05 per keypress.
> If camera FPS is too low, you can adjust the fps setting. Sometimes bandwidth or environmental factors cause the camera to auto-reduce FPS to 25.

---

## 6. Recording a Dataset

> Ensure the Pi host is running (see Section 5) before recording.
> `--arm_profile` refers to the leader arm hardware, not the follower robot.
> Replace `<Pi_IP>` with the Pi's IP address.

Before collecting, modify the leader arm port numbers in `examples/alohamini/record_bi.py`.

### AlohaMini 1 — SO-ARM leader (5-DoF) — PC

**Create a new dataset:**
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

**Resume an existing dataset (add `--resume`):**
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

### AlohaMini 2 / 2 Pro / 2 Lite — AM-ARM leader (6-DoF) — PC

First modify `examples/alohamini/record_bi.py`:

Insert this between `--arm_profile` and `--resume`:

```python
parser.add_argument("--robot_model", type=str, default="alohamini1-head",
                    choices=["alohamini1-head", "alohamini2-head", "alohamini2pro-head", "alohamini2lite-head"],
                    help="Must match the robot_model on the Pi host side")
```

Modify the `LeKiwiClientConfig` line to:

```python
robot_config = LeKiwiClientConfig(remote_ip=args.remote_ip, id=args.robot_id,
                              robot_model=args.robot_model)
```

**Create a new dataset:**
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

**Resume an existing dataset (add `--resume`):**
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
  --robot_model alohamini2lite-head \
  --resume
```

> Head pitch actions are controlled via `i`/`k` keys during recording and are automatically saved to the dataset.
> If `datasets` is missing: `pip install 'lerobot[dataset]'`
> If `socksio` is missing: `pip install 'httpx[socks]'`

---

## 7. Dataset Replay (optional)

```bash
python examples/alohamini/replay_bi.py \
  --dataset $HF_USER/am2_bi_test \
  --episode 0 \
  --remote_ip <Pi_IP>
```

---

## 8. Dataset Visualization (optional)

```bash
lerobot-dataset-viz \
  --repo-id $HF_USER/am2_bi_test \
  --episode-index 0 \
  --display-compressed-images
```

---

## 9. Training

### Local Training

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

Default `steps` is 100000. Adjust with `--steps`.

> If `accelerate` is missing: `pip install 'lerobot[training]'`

### No Local GPU?

Use any cloud GPU provider (e.g., Featurize, AutoDL, Lambda Labs, Vast.ai). Set up the environment the same way, run the same training command, then copy the checkpoint back for local evaluation.

---

## 10. Evaluation

Ensure the Pi host is running (Section 5), then run inference from the PC.

> `--robot_model` / `--robot.robot_model` **must match the model running on the Pi host:**
> - `alohamini1-head` — SO-ARM 5-DoF, **17-dim state** (includes head_pitch)
> - `alohamini2-head` / `alohamini2pro-head` / `alohamini2lite-head` — AM-ARM 6-DoF, **19-dim state** (includes head_pitch)

### Method A — evaluate_bi.py (custom script, N episodes, auto-upload to Hub)

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
  --robot_model alohamini2lite-head
```

For other models, replace `--robot_model` with: `alohamini1-head`, `alohamini2-head`, or `alohamini2pro-head`.

### Method B — lerobot-rollout (official CLI) — currently unavailable

Inference only, no recording:

```bash
python -m lerobot.scripts.lerobot_rollout \
  --strategy.type=base \
  --robot.type=alohamini_client \
  --robot.remote_ip=<Pi_IP> \
  --robot.robot_model=alohamini2lite-head \
  --policy.path=outputs/train/act_your_dataset1/checkpoints/020000/pretrained_model \
  --task="Pick and place task" \
  --fps=20 \
  --duration=45 \
  --display_data=true
```

Inference + record evaluation dataset (dataset name **must** start with `rollout_`):

```bash
python -m lerobot.scripts.lerobot_rollout \
  --strategy.type=sentry \
  --robot.type=alohamini_client \
  --robot.remote_ip=<Pi_IP> \
  --robot.robot_model=alohamini2lite-head \
  --policy.path=outputs/train/act_your_dataset1/checkpoints/020000/pretrained_model \
  --dataset.repo_id=$HF_USER/rollout_eval1 \
  --task="Pick and place task" \
  --fps=20 \
  --display_data=true
```

---

## Appendix: Hardware Configuration Table

| `--robot_model` | Follower Arm Profile | Joints/Arm | Arm Motors | Head Pitch | Base Motor | Lift Motor | Lead Screw |
|---|---|---|---|---|---|---|---|
| `alohamini1-head` | so-arm-5dof | 6 (5 joints + gripper) | All STS3215 | STS3215 (ID 12) | STS3215 | STS3215 | 84 mm/rev |
| `alohamini2-head` | am-follower-6dof | 7 (6 joints + gripper) | Mixed STS3215/STS3095 | STS3215 (ID 12) | STS3215 | STS3095 | 131 mm/rev |
| `alohamini2pro-head` | am-follower-6dof-hd | 7 (6 joints + gripper) | All STS3250 | STS3215 (ID 12) | STS3250 | STS3095 | 131 mm/rev |
| `alohamini2lite-head` | am-follower-6dof-lite | 7 (6 joints + gripper) | All STS3215 | STS3215 (ID 12) | STS3215 | STS3215 | 84 mm/rev |
