# 🎙️ Recording Data

Once you are set up with installation on your local machine, you are now ready to record data.
The following instructions are derived from https://huggingface.co/docs/lerobot/il_robots#record-a-dataset.

## 1. Main instructions

### Keyboard shortcuts during recording

Before running the recording command, it’s useful to know the available shortcuts:

- Redo a recording
  - If you consider the current episode failed or low-quality, press ← (left arrow).
  - This gives you time to reset the environment and place the robot back in its initial position.
  - Once ready, press → (right arrow) to start recording again.

- Save a recording
  - When you’re satisfied with an episode and have placed the robot in its final position, press → (right arrow).
  - This saves the episode (the saving process can take some time).
  - You then have time to reset the environment before pressing → (right arrow) again to start the next episode.

- Early stop the recording session (before all the episodes are recorded)
  - If you want to stop the recording before the total number of episodes, press `Escape`.
  - It will end the recording session, save the video and upload the dataset.
  - If you want to resume the recording in the same dataset, use `--resume=True`.

> [!NOTE]
> A practical shortcut: if you’re happy with a recording, simply double-press the right arrow. The saving process takes long enough to let you reset the environment before the next episode starts.

### Command line for recording

The command line to record:

```shell
lerobot-record \
    --robot.type=so101_follower \
    --robot.port='COM5' \
    --robot.id=follower_f0 \
    --robot.calibration_dir="path\to\lerobot-hackathon\calibration\robots\so101_follower" \
    --teleop.type=so101_leader \
    --teleop.port='COM4' \
    --teleop.id=leader_l0 \
    --teleop.calibration_dir="path\to\lerobot-hackathon\calibration\teleoperators\so101_leader" \
    --display_data=true \
    --robot.cameras="{ left: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30},  front: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30}}"  \ # <-- Change with your setting
    --dataset.root='path_to_locally_save_the_ds' \
    --dataset.repo_id='DEEL-AI/Hackathon_TeamXX' \
    --dataset.num_episodes=25 \ # Number of episodes you will record at once
    --dataset.single_task="Pick and place one green cube on the cell indicated by the card on a 2×2 grid." \ # <-- can be adapted but most follow the guidelines in the dataset section
    --dataset.push_to_hub=True \
    --resume=false  # <-- Set to true once it has been initialized
```

> [!WARNING]
> Change `--resume` to `true` **after your first recording** to append new episodes to
> the existing dataset.

> [!NOTE]
> You can change the `--dataset.single_task` to change the command prompt. For example
> "Pick and place one green cube on the top-left cell of a 2×2 grid."

### Manual upload to the Hugging Face Hub

In some environments, the automatic upload may fail because of network or certificate issues. In that case, we recommend recording the dataset locally first, then pushing it manually later.

During recording, disable the automatic upload:

```shell
lerobot-record \
    ... \
    --dataset.root="path_to_locally_save_the_ds" \
    --dataset.repo_id="DEEL-AI/Hackathon_TeamXX" \
    --dataset.push_to_hub=False
```

The dataset is still saved locally. Once the recording is finished and the network/certificate setup is fixed, push the dataset manually:

```shell
lerobot-push-dataset \
    --root="path_to_locally_save_the_ds" \
    --repo_id="DEEL-AI/Hackathon_TeamXX" \
    --private=true \
    --upload_large_folder=true
```

If you are behind a corporate network using a custom certificate authority, provide the CA bundle:

```shell
lerobot-push-dataset \
    --root="path_to_locally_save_the_ds" \
    --repo_id="DEEL-AI/Hackathon_TeamXX" \
    --private=true \
    --upload_large_folder=true \
    --ca_bundle="C:\certs\company-root-ca.pem"
```

When adding more episodes later, keep using the same local dataset root and the same Hugging Face `repo_id`, then run `lerobot-push-dataset` again. The local dataset should remain the source of truth.


## 2. Guidelines for collecting data

From this [HF blog post](https://huggingface.co/blog/lerobot-datasets) (but we changed the resolution tip):

<img width="1414" height="2000" alt="updated_dataset_recommendations" src="https://github.com/user-attachments/assets/f165feed-7ee3-4643-8cf1-f8cc4abf9fe4" />

Before recording your dataset, you can also find interesting tips in this
[blog post](https://huggingface.co/blog/sherryxychen/train-act-on-so-101).

### Available on the Hugging Face Hub

Browse all datasets: [HF Datasets](https://huggingface.co/datasets?other=LeRobot&sort=trending)

**Visualize a LeRobot Dataset**
Use the interactive tool: [HF Robot Viz Space](https://huggingface.co/spaces/lerobot/visualize_dataset)

You can also use the local commands as described here:

- Visualize data stored on a local machine:

```shell
local$ lerobot-dataset-viz \
    --repo-id lerobot/pusht \
    --episode-index 0
```

- Visualize data stored on a distant machine with a local viewer:

```shell
distant$ lerobot-dataset-viz \
    --repo-id lerobot/pusht \
    --episode-index 0 \
    --save 1 \
    --output-dir path/to/directory

local$ scp distant:path/to/directory/lerobot_pusht_episode_0.rrd .
local$ rerun lerobot_pusht_episode_0.rrd
```

- Visualize data stored on a distant machine through streaming:
(You need to forward the websocket port to the distant machine, with
`ssh -L 9087:localhost:9087 username@remote-host`)

```shell
distant$ lerobot-dataset-viz \
    --repo-id lerobot/pusht \
    --episode-index 0 \
    --mode distant \
    --ws-port 9087

local$ rerun ws://localhost:9087
```
