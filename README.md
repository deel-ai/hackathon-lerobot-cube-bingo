# 🤖 ISAE-SUPAERO AIBT HACKATHON 2026

Welcome to the ISAE-SUPAERO AIBT HACKATHON 2026 repository 🎉

Here you’ll find everything you need to get started: objectives, rules, tips, and resources to make the most out of this exciting challenge!

<a id="top"></a>

<details>
<summary><strong>📚 Table of contents</strong></summary>

- [🤖 ISAE-SUPAERO AIBT HACKATHON 2026](#-isae-supaero-aibt-hackathon-2026)
  - [🎯 Task Overview](#-task-overview)
    - [🏆 Evaluation Rules \& Scoring](#-evaluation-rules--scoring)
    - [🗓️ Schedule recommendations](#️-schedule-recommendations)
    - [🔍 Tips \& Recommendations](#-tips--recommendations)
    - [✅ Key Takeaways](#-key-takeaways)
  - [⚙️ 2. Setup Instructions](#️-2-setup-instructions)
  - [🎙️ 3. Data recording](#️-3-data-recording)
  - [⏱️ 4. Model training](#️-4-model-training)
  - [5. Inference](#5-inference)

</details>

## 🎯 Task Overview

The challenge: teach a robot arm to place 1, 2, or 3 colored cubes onto a 2×2 wooden grid. Each cube should be placed in the cell of the corresponding colored cross.

Sounds simple? In practice, it’s a tough robotics + learning task! Success depends on how strategically you build datasets (coverage & curriculum), train models, and plan your approach.

### 🏆 Evaluation Rules & Scoring

Your robot will be evaluated over 4 levels of increasing difficulty.
Each level has 5 trials. You must succeed on at least 3/5 trials to unlock the next level.

<img width="600" alt="hackathon levels" src="assets/hackathon.drawio.png" />

**Level 1 — Single Color, Fixed Position**

Task: Pick up one cube of a single color and place it on the same target cell every time (as indicated by the card) in ≤ 20s.
Scoring: 5 points per success (+5 bonus if the cube is grasped on the first attempt).

**Level 2 — Single Cube, Varying Positions**

Task: Pick up one cube and place it on the correct cell (position varies across trials) in ≤ 20s.
Scoring: 10 points per success (+5 bonus for first-attempt grasps).

**Level 3 — Two Colored Cubes, Varying Positions**

Task: Pick and place 2 cubes of different colors on their respective target cells in ≤ 30s.
Scoring: 30 points per success.

**Level 4 — Three Colored Cubes, Varying Positions**

Task: Pick and place 3 cubes of different colors on their respective target cells in ≤ 30s.
Scoring: 50 points per success.

> [!WARNING]
> ⚠️ Reality check: With only 2 days, even reaching Level 2 is a big success!

### 🗓️ Schedule recommendations

**Day 1**

Morning: 
- Use the morning to discuss strategy BEFORE starting to record data. 
- Split the work: one or two to get the data recording, one or two setting the cloud environment (see dedicated session), one to test that inference is working.
- Even though it is on a few data points test your end-to-end pipeline

Afternoon:
- Record, record and record!
- Try trainings and evaluations as tonight you will launch one or two BIG training.

17:30: First evaluation attempt (even with partial training).

**Day 2**

09:00: Evaluation with overnight training results on first levels.
09:30 - 11:00: Try to adjust and launch further fine-tuning
11:00 - 16:00: Preparation of team presentations (strategies, results, lessons learned)
16:00-17:00: Final wrap-up session of the hackathon.

### 🔍 Tips & Recommendations

Plan your recording setup: Each team gets 2 external cameras. Place them wisely (wrist, top, side view, etc.). Name them clearly when recording datasets.

Record plenty of data: Rotate who records to get variety. Consistency matters!

Mark positions: If you set up a controlled environment, mark object positions (objects may be moved overnight).

**Robustness vs. simplicity**:

- Controlled environment = easier early progress.
- Varied environments = robustness but needs more data.
- Curriculum strategy: Start with lots of Level 1 data, then gradually add Level 2, Level 3, etc. However, this is not an obligation. If you and your teammates want to win all or nothing we respect that!

> [!NOTE]
>💡 Check out the [dataset guidelines](#guidelines-for-collecting-data) for more details.


### ✅ Key Takeaways

- Think strategically: scoring is incremental, don’t rush for Level 4.
- Save checkpoints and test early & often.
- Teamwork matters as much as models: rotate recording, share insights.
- Remember: the goal is to learn, experiment, and have fun 🎉

## ⚙️ 2. Setup Instructions

Two environments should be set:

- one on your local machine for data recording and inference,
- a second one on a remote GPU machine for model training.

> [!NOTE]
> If you have computers that possess a good GPU, you can consider using it for training. However, while training you might not be > able to use it to do other things. Therefore, prefer using the GCP solution.

Follow the setup instructions for both systems in [setup_instructions.md](./setup_instructions.md).

## 🎙️ 3. Data recording

When the setup is ready on your local machine, you can start recording episodes for your
dataset. Follow the instructions and tips in [data_recording.md](./data_recording.md).

## ⏱️ 4. Model training

Once data is recorded, you can train a model on the remote machine. We provide a toy
dataset to try a training stage without waiting for your own data recording. You can
save time to ensure that your remote machine is correctly set up and ready for training.

The toy dataset can be found [here on Hugging Face](https://huggingface.co/datasets/DEEL-AI/Hackathon_Team0Z)
and can be used in the training command line using argument
`--dataset.repo_id=DEEL-AI/Hackathon_Team0Z`.

All instructions and tips for training are given in [training.md](./training.md).

## 5. Inference

To run inference on your local machine, you first need to get your trained checkpoints.

We suppose you are in one of the following two cases:

### 5.1 You uploaded your policy on HF

```shell
lerobot-record  \
    --robot.type=so101_follower \
    --robot.port='COM5' \
    --robot.id=follower_f0 \
    --robot.calibration_dir="path\to\lerobot-hackathon\calibration\robots\so101_follower" \
    --robot.cameras="{ left: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30},  front: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30}}"  \ # <-- Change with your setting
    --teleop.type=so101_leader \
    --teleop.port='COM4' \
    --teleop.id=leader_l0 \
    --teleop.calibration_dir="path\to\lerobot-hackathon\calibration\teleoperators\so101_leader" \
    --display_data=true \
    --dataset.single_task="Pick and place one green cube on the cell indicated by the card on a 2×2 grid." \
    --dataset.root='./eval_hackathon_9_cubes_v1' \
    --dataset.repo_id='DEEL-AI/eval_Hackathon_TeamXX' \
    --dataset.push_to_hub=false \
    --policy.repo_id=Kyumeo/ACT_Test \ #CHANGEME
    --policy.type=act 
```

> [!NOTE]
> You can test your inference setup using the default policy provided in the previous shell. It won't be a good one but it will be enough to validate that your environment is working!

You don’t need to include the teleop arguments if you prefer not to. However, adding them allows you to press ← (left arrow) during inference to temporarily take manual control of the robot and reset it, before pressing → (right arrow) to continue to the next episode.

> [!NOTE]
> You can modify the --dataset.single_task flag to change the command prompt. That said, we recommend using the exact same commands as those in your dataset to ensure consistency.

The keyboard shortcuts behave the same way as during recording, except you won’t need to teleoperate—the robot will autonomously execute episodes.
For testing, we provide a sample policy that you can run before training your own. Its behavior may be erratic, but it’s useful for verifying that inference is working correctly.

### 5.2 You did not uploaded your policy on HF

If you want to get them from the remote machine, you can use scp:

```shell
scp -r username@remote_machine://output_dir/train/act_so101_test/checkpoints/last C:\path\to\outputs\train\act_so101_test\checkpoints
```

Then, using your local environment setup, you can run:

```shell
lerobot-record  \
    --robot.type=so101_follower \
    --robot.port='COM5' \
    --robot.id=follower_f0 \
    --robot.calibration_dir="path\to\lerobot-hackathon\calibration\robots\so101_follower" \
    --robot.cameras="{ left: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30},  front: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30}}"  \ # <-- Change with your setting
    --teleop.type=so101_leader \
    --teleop.port='COM4' \
    --teleop.id=leader_l0 \
    --teleop.calibration_dir="path\to\lerobot-hackathon\calibration\teleoperators\so101_leader" \
    --display_data=true \
    --dataset.single_task="Pick and place one green cube on the cell indicated by the card on a 2×2 grid." \
    --dataset.root='./eval_hackathon_9_cubes_v1' \
    --dataset.repo_id='DEEL-AI/eval_Hackathon_TeamXX' \
    --dataset.push_to_hub=false \
    --policy.path="C:\path\to\outputs\train\act_so101_test\checkpoints\last\pretrained_model"
```

You don’t need to include the teleop arguments if you prefer not to. However, adding them allows you to press ← (left arrow) during inference to temporarily take manual control of the robot and reset it, before pressing → (right arrow) to continue to the next episode.

> [!NOTE]
> You can modify the --dataset.single_task flag to change the command prompt. That said, we recommend using the exact same commands as those in your dataset to ensure consistency.

The keyboard shortcuts behave the same way as during recording, except you won’t need to teleoperate—the robot will autonomously execute episodes.
For testing, we provide a sample policy that you can run before training your own. Its behavior may be erratic, but it’s useful for verifying that inference is working correctly.

✅ You are now fully set up and ready to record, train, and run inference!
