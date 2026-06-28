# ⏱️ Model training

You will find here all the instructions and tips for training a model on a remote
machine. Before training, make sure that your remote machine is correctly set up,
following [setup_instructions.md](./setup_instructions.md).

## 1. Get your dataset

If you uploaded your dataset on Hugging Face, make sure to be authenticated on your
remote machine with a token. 

```
hf auth login
```

You can then directly use your repo id in the command line `lerobot-train`. You can also manually download it using:

```shell
hf download DEEL-AI/Hackathon_TeamXX --repo-type dataset
```

If your dataset is only stored locally, you can copy it on your remote machine using
`scp`. Make sure to copy all the subdirectories containing metadata, videos, parquet
files, etc.

For a better understanding of the `LeRobotDataset`, you can read the
[description of v3.0 dataset format](https://huggingface.co/docs/lerobot/lerobot-dataset-v3).

## 2. Training with ACT (Action Chunking Transformer)

### 2.1 Authenticate with W&B

If you don't have one create a free W&B account for logging and monitoring training. Once you have a token you can authenticate using:

```
wandb login
```

LeRobot comes with a handy command line to train an ACT model:

```shell
lerobot-train \
  --dataset.repo_id=DEEL-AI/Hackathon_TeamXX \ # CHANGEME
  --policy.type=act \
  --output_dir=/output_dir_with_space/train/act_so101_test \
  --job_name=act_so101_test \
  --policy.device=cuda \
  --policy.push_to_hub=true \
  --policy.repo_id=Username/ACT_Test \ # CHANGEME
  --batch_size=64 \
  --save_freq=100 \
  --steps=1000 \
  --wandb.enable=true \
  --wandb.disable_artifact=true
```

As a test you can run the following script with `dataset.repo_id=DEEL-AI/Hackathon_Team0Z`. (Don't forget to download it with `hf download DEEL-AI/Hackathon_Team0Z --repo-type dataset`)

We set `--policy.push_to_hub` to true as it will ease the process of uploading model's checkpoints from GCP instances (or any other remote instances) and download them back for [inference](./README.md#5-inference).

For this test I voluntarily set a low number of steps and a saving frequency that is low. **However, having such a low frequency will fill up space quickly so be mindful of this parameter**.

ACT come with default values for `steps` and `save_freq`. It is up to you to check if the are those you want to use.

If you want to resume a training:

```shell
lerobot-train \
  --config_path=/output_dir_with_space/train/outputs/act_so101_test/checkpoints/last/pretrained_model/train_config.json \
  --resume=true \
  --policy.device=cuda \
  --policy.push_to_hub=true \
  --policy.repo_id=Username/ACT_Test \ # CHANGEME
  --steps=200000 \
  --wandb.enable=true \
  --wandb.disable_artifact=true
```

> [!WARNING]
> Here `--steps` is not the number of additional steps you want to do. It is the number of steps it must reach from the CKPT step you provided.


## 3. Training with SmolVLA

Additionnally to finetune SmolVLA:

```shell
pip install -e ".[smolvla]"
```

Finally **check your pytorch version** and use `set_cuda_version` with matching distributions.

Then:

```shell
lerobot-train \
  --policy.path=cijerezg/smolvla-test \
  --dataset.repo_id=DEEL-AI/Hackathon_TeamXX \
  --output_dir=/output_dir_with_space/train/smolvla_so101_test \
  --job_name=smolvla_so101_test \
  --policy.device=cuda \
  --policy.push_to_hub=false \
  --batch_size=64 \
  --steps=20000 \
  --save_freq=5000 \
  --wandb.enable=true \
  --wandb.disable_artifact=true
```

As a test you can run the following script with `dataset.repo_id=DEEL-AI/Hackathon_Team0Z`

To resume training it is the same as for ACT:

```shell
lerobot-train \
  --config_path=/output_dir_with_space/train/outputs/smolvla_so101_test/checkpoints/last/pretrained_model/train_config.json \
  --resume=true \
  --policy.device=cuda \
  --policy.push_to_hub=false \
  --steps=40000 \
  --wandb.enable=true \
  --wandb.disable_artifact=true
```

## 4. Training Time Examples

- ACT (RTX 4090 - 24GB VRAM): batch size 32, 100k steps → ~7h
- SmolVLA (RTX 4090 - 24GB VRAM): batch size 64, 20k steps → ~7 h
- Pi0.5: TBD

> [!NOTE]
> ⚠️ Loss curves may not always reflect real-world performance. Save checkpoints often and test them on the robot!
