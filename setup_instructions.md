# ⚙️ Setup Instructions

This guide explains how to prepare your environment. Two environments are described:

1. On a local (Windows) machine, for data recording and inference,
2. On a remote (Linux) machine with GPU, for model training.
3. On a GCP VM, for model training. (RECOMMENDED FOR THE TRAINING)

## 📂 1. Source Code

For the hackathon, we provide a dedicated fork of Hugging Face’s LeRobot v0.4.3 repository.

👉 Use the [custom repository](https://github.com/deel-ai/lerobot-hackathon/tree/hackathon-sdd-2026) as your codebase.

This custom repo will be used for both your local and remote machines. See below for
download and installation.

## 🖥️ 2. Setup environment on your local Windows machine

### 1. Install FFmpeg

Open PowerShell and run:

```shell
winget install ffmpeg
```

### 2. Clone the Repository

```shell
mkdir hackathon
cd hackathon
git clone --single-branch -b hackathon-sdd-2026 git@github.com:deel-ai/lerobot-hackathon.git
```

### 3. Create and Activate Virtual Environment

Choose your favorite virtual environment (conda, venv, uv). For conda:

```shell
conda create -y -n lerobot python=3.11
conda activate lerobot
```

For uv:

```shell
uv venv --python 3.11
# activate your venv based on your OS
.venv\Scripts\activate  # e.g. for Windows Powershell
```

### 4. Install PyTorch with the correct CUDA version

First, check your CUDA version using the command:

```bash
nvidia-smi
```

Then, install torch==2.7.1 and torchvision==0.22.1 for the corresponding CUDA version by following the official [PyTorch installation guide](https://pytorch.org/get-started/previous-versions/).

For example, if your CUDA version is 12.6, run:

```bash
pip install torch==2.7.1 torchvision==0.22.1 --index-url https://download.pytorch.org/whl/cu126
```

### 5. Install LeRobot (from custom repo) and other dependencies

```shell
pip install -e lerobot-hackathon
pip install -e "lerobot-hackathon[feetech]"
```

## 🔧 3. Calibrate the robot arms

Calibration ensures that leader and follower arms map correctly. A well-calibrated robot allows a model trained on one setup to generalize to another.
These instructions follow the original LeRobot [S0-101 documentation](https://huggingface.co/docs/lerobot/so101).

### 1. Connect the arms

> [!WARNING]
> Power supply is 12V for the Follower and 5V for the Leader. Be cautious to not mistake one with the other. If in doubt ask us!

- Power both arms.
- Use **USB-C → USB-A** cables to connect arms to your PC.
- Always use the same USB ports for consistency.

Find the port names:

```shell
lerobot-find-port
```

Example output:

```shell
Finding all available ports for the MotorBus.
['COM4', 'COM5']
Remove the USB cable from your MotorsBus and press Enter when done.

[...Disconnect corresponding leader or follower arm and press Enter...]

The port of this MotorsBus is 'COM4'
Reconnect the USB cable.
```

In our example, if you disconnected the leader arm, then the leader arm is on `'COM4'` and the follower `'COM5'`.

Remember it, as it will be important when you will operate.

### 2. Calibrate the Follower arm

> [!NOTE]
> Using Powershell you should replace the " \ " with "`", and using the shell with "^"

**Classical shell**
```shell
lerobot-calibrate \
    --robot.type=so101_follower \
    --robot.port='COM5' \ # <- The port of your robot
    --robot.id=follower_idontheetiquette  \ # <- Give the robot a unique name
    --robot.calibration_dir="path\to\lerobot-hackathon\calibration\robots\so101_follower"
```

**Windows Command shell**
```shell
lerobot-calibrate ^
    --robot.type=so101_follower ^
    --robot.port='COM5' ^ # <- The port of your robot
    --robot.id=follower_idontheetiquette  ^ # <- Give the robot a unique name
    --robot.calibration_dir="path\to\lerobot-hackathon\calibration\robots\so101_follower"
```

**PowerShell**
```shell
lerobot-calibrate `
    --robot.type=so101_follower `
    --robot.port='COM5' ` # <- The port of your robot
    --robot.id=follower_idontheetiquette  ` # <- Give the robot a unique name
    --robot.calibration_dir="path\to\lerobot-hackathon\calibration\robots\so101_follower"
```

> [!WARNING]
> **Move joints slowly during calibration.** Fast manual motions can trigger motor faults or overheating protection, which may later cause detection issues during teleop/record/inference.

For the calibration itself, the easiest is to follow the video in the dedicated section in this [tutorial](https://huggingface.co/docs/lerobot/so101#calibrate).

Your file will be saved in your calibration dir with the name "follower_idontheetiquette.json".
This is interesting as once a calibration has been properly done you can share it with member of your teams without the need of doing the calibration again.

### 3. Calibrate the Leader arm

```shell
lerobot-calibrate \
    --teleop.type=so101_leader \
    --teleop.port='COM4' \ # <- The port of your robot
    --teleop.id=leader_idontheetiquette \ # <- Give the robot a unique name
    --teleop.calibration_dir="path\to\lerobot-hackathon\calibration\teleoperators\so101_leader"
```

> [!NOTE]
> The `--robot` flag is for the **follower** and `--teleop` for the **leader** arm.

### 4. Test calibration with teleoperation

While having your hands on the **leader** arm:

```shell
lerobot-teleoperate \
    --robot.type=so101_follower \
    --robot.port="COM5" \ #CHANGEME
    --robot.id="follower_f0" \ #CHANGEME
    --robot.calibration_dir="path\to\lerobot-hackathon\calibration\robots\so101_follower" \ #CHANGEME
    --teleop.type=so101_leader \
    --teleop.port="COM4" \ #CHANGEME
    --teleop.id="leader_l0" \ #CHANGEME
    --teleop.calibration_dir="path\to\lerobot-hackathon\calibration\teleoperators\so101_leader" \ #CHANGEME
    --display_data=true
```

A `rerun.io` window should open. The follower arm must match the leader precisely. In case of lags, trying killing unnecessary processes.
If it is still not working → **redo calibration**.

> [!WARNING]
> **Troubleshooting — Motors blinking red / not detected.**
>
> If a motor LED blinks red or a motor is not detected when starting teleoperation, recording, or inference, it may be an overheating protection state (see related discussion in the LeRobot repo, [issue #441](https://github.com/huggingface/lerobot/issues/441)).
>
> **Quick fix that worked for us:**
>
> 1. Power off the affected arm (disconnect the arm’s power).
> 2. Wait a few seconds.
> 3. Power it back on and retry.

## 🎥 4. Test cameras

### 1. Plug cameras into your computer.

A dock might be needed to have enough port and to have your computer charging.

### 2. Identify them:

```shell
lerobot-find-cameras opencv
```

Example output:

```shell
--- Detected Cameras ---
Camera #0:
  Name: OpenCV Camera @ 0
  Type: OpenCV
  Id: 0
  Backend api: AVFOUNDATION
  Default stream profile:
    Format: 16.0
    Width: 1920
    Height: 1080
    Fps: 15.0
--------------------
(more cameras ...)
```

> [!NOTE]
> This identifier might change if you reboot your computer or re-plug your camera, a behavior mostly dependant on your operating system.

### 3. Match IDs with physical placement by checking saved images in:

```shell
repository_dir/outputs/captured_images
```

Let's say that `opencv_0.png` is a camera positioned on the left of our robot and `opencv_2.png` correspond to a camera positioned
in front of it.

### 4. Test teleoperation with two cameras:

```shell
lerobot-teleoperate \
    --robot.type=so101_follower \
    --robot.port="COM5" \
    --robot.id="follower_f0" \
    --robot.calibration_dir="path\to\lerobot-hackathon\calibration\robots\so101_follower" \
    --robot.cameras="{ left: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30},  front: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30}}" \ # <-- Your setting
    --teleop.type=so101_leader \
    --teleop.port="COM4" \
    --teleop.id="leader_l0" \
    --teleop.calibration_dir="path\to\lerobot-hackathon\calibration\teleoperators\so101_leader" \
    --display_data=true
```

> [!TIP]
> **Performance & latency:** We strongly recommend 480p (width=640, height=480) for each camera.
> With 2 cameras at ≥720p, we observed choppy teleoperation/inference that severely hurts success rates. 480p keeps streams smooth while preserving enough detail for the task.

You should now "see" what your robots is seeing in the `rerun.io` window. It will be useful to use that vision before actually recording data to place them as you wish.


## 🖥️ 5. Setup environment on a remote (Linux) machine

### 1. Create environment (here with conda, but you can use venv or uv)

```shell
conda create -y -n lerobot python=3.10
conda activate lerobot
```

### 2. Install FFmpeg

```shell
conda install ffmpeg=7.1.1 -c conda-forge
```

Checks that when you do:

```shell
which ffmpeg
```

Outputs looks like:

```shell
/home/username/.conda/envs/lerobot/bin/ffmpeg
```

And that you see `libsvtav1` in the list of the output of the following command:

```shell
ffmpeg -encoders
```

### 3. Clone & install repo

```shell
mkdir hackathon
cd hackathon
git clone --single-branch -b hackathon-sdd-2026 git@github.com:deel-ai/lerobot-hackathon.git
```

### 4. Install PyTorch with the correct CUDA version

First, check your CUDA version using the command:

```bash
nvidia-smi
```

Then, install torch==2.7.1 and torchvision==0.22.1 for the corresponding CUDA version by following the official [PyTorch installation guide](https://pytorch.org/get-started/previous-versions/).

For example, if your CUDA version is 12.6, run:

```bash
pip install torch==2.7.1 torchvision==0.22.1 --index-url https://download.pytorch.org/whl/cu126
```

### 5. Install LeRobot (from custom repo)

```shell
pip install -e lerobot-hackathon
```

### 6. Authenticate on Hugging Face (if needed)

```shell
hf auth login
wandb login
```

> [!NOTE]
> Make sure that you use a token with the right permissions

## 🖥️ 4. Build a GCP VM for training

### 1. Ask to be added to the hackathon project!

You will need a google account to do so!

### 2. Find a VM that can train the model you want

The first step is to instantiate a VM which have proportionate hardware for your training.

We believe that in only 2 days the most reasonnable approach is to focus on the ACT policy training.

> [!WARNING]
> If you want to do differently come and validate with us your approach. Indeed, we have a total budget that we cannot exceed.
> However, this budget should allow some freedom so don't hesitate and ask.

For ACT training and in our case the recommended setup is to use a L4 GPU with a g2-standard-8 setting. The first step is to build a VM with such hardware and the first challenge is finding it. As availability is not alway easy we prepared a script that you can run to automatically try all zones with this hardware and try to build it.

**Europe First**

```shell
zones=(  
europe-west1-b europe-west1-c  
europe-west2-a europe-west2-b  
europe-west3-a europe-west3-b  
europe-west4-a europe-west4-b europe-west4-c  
europe-west6-b europe-west6-c  
)  
  
for zone in "${zones[@]}"; do  
  echo "Testing $zone"
  gcloud compute instances create act-test \
    --zone="$zone" \
    --machine-type=g2-standard-8 \
    --accelerator=type=nvidia-l4,count=1 \
    --image-family=ubuntu-2204-lts \
    --image-project=ubuntu-os-cloud \
    --boot-disk-size=200GB \
    --maintenance-policy=TERMINATE \
    --quiet && echo "SUCCESS in $zone" && break
done
```

**America Then (If None in Europe)**

```shell
for zone in us-central1-a us-central1-b us-central1-c \
            us-east1-b us-east1-c us-east1-d \
            us-west1-a us-west1-b us-west1-c; do
  echo "Testing $zone"

  gcloud compute instances create act-test \
    --zone="$zone" \
    --machine-type=g2-standard-8 \
    --accelerator=type=nvidia-l4,count=1 \
    --image-family=ubuntu-2204-lts \
    --image-project=ubuntu-os-cloud \
    --boot-disk-size=200GB \
    --maintenance-policy=TERMINATE \
    --quiet && echo "SUCCESS in $zone" && break
done
```

### 3. Connect and build dependencies

Connect to your instantiated VM through SSH with the Google Cloud Console. 

**Install NVIDIA Driver**

```shell
sudo apt update
sudo apt install -y nvidia-driver-580-server
sudo reboot
```

After reconnecting:

```
nvidia-smi
```

You should see something like:

```
NVIDIA L4Driver Version: 580.xxxCUDA Version: 12.x
```

**FFMPEG**

```
sudo apt install -y ffmpeg libavcodec-dev libavformat-dev libavutil-dev libswscale-dev libavfilter-dev libavdevice-dev
```

**UV**

```
curl -LsSf https://astral.sh/uv/install.sh | sh

source $HOME/.local/bin/env
```

**Clone the custom repository**

Create a ssh key on the VM instance:

```
ssh-keygen -t ed25519 -C "lucas.hervier@irt-saintexupery.com"
```

Copy the public key and add it to your github account:

```
cat ~/.ssh/id_ed25519.pub
```

Then:

```
mkdir hackathon
cd hackathon
git clone --single-branch -b hackathon-sdd-2026 git@github.com:deel-ai/lerobot-hackathon.git
```

**Create Python env**

```
uv venv --python 3.12
source .venv/bin/activate

uv pip install torch torchvision torchaudio --index-url https://download.pytorch.org/whl/cu129

uv pip install -e lerobot-hackathon
```

Once this is done you can move on to the [training section](training.md) where a dry-run test is available.
