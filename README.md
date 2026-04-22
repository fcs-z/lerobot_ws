# lerobot_ws
```
X86 Ubuntu 22.04
```
## 1、Environment
- Install miniconda3
```
mkdir -p ~/miniconda3
cd miniconda3
wget https://repo.anaconda.com/miniconda/Miniconda3-latest-Linux-x86_64.sh -O ~/miniconda3/miniconda.sh
bash ~/miniconda3/miniconda.sh -b -u -p ~/miniconda3
rm ~/miniconda3/miniconda.sh
source ~/miniconda3/bin/activate
conda init --all
```
- Create conda env and activate it
```
conda create -n env_lerobot python=3.10 && conda activate env_lerobot
```
- Git clone
```
git clone https://github.com/Seeed-Projects/lerobot.git ~/lerobot
```
- Install ffmpeg
```
conda install ffmpeg=7.1.1 -c conda-forge
```
- Install dependencies(feetech)
```
cd ~/lerobot && pip install -e ".[feetech]"
```
- Check Pytorch and Torchvision
```
import torch
print(torch.cuda.is_available())
```
## 2、Record steering gear
The driver board is connected to the computer using a USB cable, and the driver board is connected to the power supply
- Find usb port
```
lerobot-find-port
```
Unplug the USB cable, record the port: /dev/ttyACM0, and grant port permissions
```
sudo chmod 666 /dev/ttyACM0
```
- Calibration (taking follower as an example)

The driving plate is connected with the steering gear from the claw to the base in turn
```
lerobot-setup-motors \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM0
```
After pressing Enter, the drive board continues to connect to the next steering gear
- The leader is the same
```
lerobot-setup-motors \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM0
```
## 3、Calibrate steering gear
Expand the robotic arm to the corresponding position (the base is perpendicular to the desktop, and the remaining robotic arm is horizontal to the desktop, with an angle of 45° upward)
- follower
```
lerobot-calibrate \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM0 \
    --robot.id=my_awesome_follower_arm
```
Rotate the robot arm and record the maximum and minimum angles of the robot arm
- leader
```
lerobot-calibrate \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM1 \
    --teleop.id=my_awesome_leader_arm
```
## 4、Remote control
```
sudo chmod 666 /dev/ttyACM*
```
```
lerobot-teleoperate \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM0 \
    --robot.id=my_awesome_follower_arm \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM1 \
    --teleop.id=my_awesome_leader_arm
```
## 5、Use camera
find camera：
```
lerobot-find-cameras opencv 
```
one camera:
```
lerobot-teleoperate \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM0 \
    --robot.id=my_awesome_follower_arm \
    --robot.cameras='{ front: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30, fourcc: "MJPG"} }' \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM1 \
    --teleop.id=my_awesome_leader_arm \
    --display_data=true
```
two cameras:
```
lerobot-teleoperate \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM0 \
    --robot.id=my_awesome_follower_arm \
    --robot.cameras='{ front: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30, fourcc: "MJPG"}, side: {type: opencv, index_or_path: 4, width: 640, height: 480, fps: 30, fourcc: "MJPG"} }' \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM1 \
    --teleop.id=my_awesome_leader_arm \
    --display_data=true
```
## 6、make a dataset
- save dataset to local:
```
lerobot-record \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM0 \
    --robot.id=my_awesome_follower_arm \
    --robot.cameras='{ front: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30, fourcc: "MJPG"}, side: {type: opencv, index_or_path: 4, width: 640, height: 480, fps: 30, fourcc: "MJPG"} }' \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM1 \
    --teleop.id=my_awesome_leader_arm \
    --display_data=true \
    --dataset.repo_id=lerobot_dataset/test01 \
    --dataset.num_episodes=5 \
    --dataset.single_task="Grab the cube" \
    --dataset.push_to_hub=false \
    --dataset.episode_time_s=30 \
    --dataset.reset_time_s=30 
```
If the recording is interrupted, you can resume it by running the same command again with --resume=true.
```
--resume=true
```
During recovery, set --dataset.num_episodes to the number of episodes to be recorded additionally (not the total number of episodes in the dataset).

replay:
```
lerobot-replay \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM0 \
    --robot.id=my_awesome_follower_arm \
    --dataset.repo_id=lerobot_dataset/test01 \
    --dataset.episode=0
```
- save dataset to Hugging Face Hub:
```
huggingface-cli login --token ${HUGGINGFACE_TOKEN} --add-to-git-credential
```
```
HF_USER=$(huggingface-cli whoami | head -n 1)
echo $HF_USER
```
```
lerobot-record \
    --robot.type=so101_follower \
    --robot.port=/dev/ttyACM0 \
    --robot.id=my_awesome_follower_arm \
    --robot.cameras='{ front: {type: opencv, index_or_path: 0, width: 640, height: 480, fps: 30, fourcc: "MJPG"}, side: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30, fourcc: "MJPG"} }' \
    --teleop.type=so101_leader \
    --teleop.port=/dev/ttyACM1 \
    --teleop.id=my_awesome_leader_arm \
    --display_data=true \
    --dataset.repo_id=${HF_USER}/record-test \
    --dataset.num_episodes=5 \
    --dataset.single_task="Grab the cube" \
    --dataset.push_to_hub=true \
    --dataset.episode_time_s=30 \
    --dataset.reset_time_s=30 
```
## 7、train & eval
### 7.1 ACT
- train(local)
```
lerobot-train \
  --dataset.repo_id=lerobot_dataset/test01 \
  --policy.type=act \
  --output_dir=outputs/train/act_so101_test01 \
  --job_name=act_so101_test01 \
  --policy.device=cuda \
  --wandb.enable=false \
  --policy.push_to_hub=false\
  --steps=300000 
```
- eval(local)
```
lerobot-record \
  --robot.type=so101_follower \
  --robot.port=/dev/ttyACM0 \
  --robot.cameras='{ front: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30, fourcc: "MJPG"}, side: {type: opencv, index_or_path: 4, width: 640, height: 480, fps: 30, fourcc: "MJPG"} }' \
  --robot.id=my_awesome_follower_arm \
  --display_data=false \
  --dataset.repo_id=lerobot_dataset/eval_test01 \
  --dataset.single_task="Put lego brick into the transparent box" \
  --policy.path=outputs/train/act_so101_test01/checkpoints/last/pretrained_model
```
- train(Hugging Face Hub)
```
lerobot-train \
  --dataset.repo_id=${HF_USER}/so101_test \
  --policy.type=act \
  --output_dir=outputs/train/act_so101_test \
  --job_name=act_so101_test \
  --policy.device=cuda \
  --wandb.enable=false \
  --steps=300000 
```
### 7.2 SmolVLA
- train(Hugging Face Hub)
```
lerobot-train \
  --policy.path=lerobot/smolvla_base \
  --dataset.repo_id=${HF_USER}/mydataset \
  --batch_size=64 \
  --steps=20000 \
  --output_dir=outputs/train/my_smolvla \
  --job_name=my_smolvla_training \
  --policy.device=cuda \
  --wandb.enable=true
```
- eval(Hugging Face Hub)
```
lerobot-record \
  --robot.type=so101_follower \
  --robot.port=/dev/ttyACM0 \
  --robot.id=my_blue_follower_arm \
  --robot.cameras='{ front: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30, fourcc: "MJPG"}, side: {type: opencv, index_or_path: 4, width: 640, height: 480, fps: 30, fourcc: "MJPG"} }' \
  --dataset.single_task="Grasp a lego block and put it in the bin." \
  --dataset.repo_id=${HF_USER}/eval_DATASET_NAME_test \
  --dataset.episode_time_s=50 \
  --dataset.num_episodes=10 \
  --policy.path=${HF_USER}/FINETUNE_MODEL_NAME
```
### 7.3 Pi0
- train(Hugging Face Hub)
```
lerobot-train \
  --policy.type=pi0 \
  --dataset.repo_id=${HF_USER}/my_dataset \
  --job_name=pi0_training \
  --output_dir=outputs/pi0_training \
  --policy.pretrained_path=lerobot/pi0_base \
  --policy.repo_id=${HF_USER}/my_pi0_policy \
  --policy.compile_model=true \
  --policy.gradient_checkpointing=true \
  --policy.dtype=bfloat16 \
  --policy.freeze_vision_encoder=false \
  --policy.train_expert_only=false \
  --steps=3000 \
  --policy.device=cuda \
  --batch_size=32 \
  --wandb.enable=false 
```
- eval(Hugging Face Hub)
```
lerobot-record \
  --robot.type=so101_follower \
  --robot.port=/dev/ttyACM0 \
  --robot.cameras='{ front: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30, fourcc: "MJPG"}, side: {type: opencv, index_or_path: 4, width: 640, height: 480, fps: 30, fourcc: "MJPG"} }' \
  --robot.id=my_awesome_follower_arm \
  --display_data=false \
  --dataset.repo_id=${HF_USER}/eval_my_pi0_test \
  --dataset.single_task="Put lego brick into the transparent box" \
  --dataset.episode_time_s=50 \
  --dataset.num_episodes=10 \
  --policy.path=outputs/pi0_training/checkpoints/last/pretrained_model
```
### 7.4 Pi0.5
- train(Hugging Face Hub)
```
lerobot-train \
  --dataset.repo_id=${HF_USER}/my_dataset \
  --policy.type=pi05 \
  --output_dir=outputs/pi05_training \
  --job_name=pi05_training \
  --policy.repo_id=${HF_USER}/my_pi05_policy \
  --policy.pretrained_path=lerobot/pi05_base \
  --policy.compile_model=true \
  --policy.gradient_checkpointing=true \
  --policy.dtype=bfloat16 \
  --policy.freeze_vision_encoder=false \
  --policy.train_expert_only=false \
  --steps=3000 \
  --policy.device=cuda \
  --batch_size=32 \
  --wandb.enable=false
```
- eval(Hugging Face Hub)
```
lerobot-record \
  --robot.type=so101_follower \
  --robot.port=/dev/ttyACM0 \
  --robot.cameras='{ front: {type: opencv, index_or_path: 2, width: 640, height: 480, fps: 30, fourcc: "MJPG"}, side: {type: opencv, index_or_path: 4, width: 640, height: 480, fps: 30, fourcc: "MJPG"} }' \
  --robot.id=my_awesome_follower_arm \
  --display_data=false \
  --dataset.repo_id=${HF_USER}/eval_my_pi05_test \
  --dataset.single_task="Put lego brick into the transparent box" \
  --dataset.episode_time_s=50 \
  --dataset.num_episodes=10 \
  --policy.path=outputs/pi05_training/checkpoints/last/pretrained_model
```
### 7.5 GROOT N1.5
- train
```
accelerate launch \
  --multi_gpu \
  --num_processes=$NUM_GPUS \
  $(which lerobot-train) \
  --output_dir=$OUTPUT_DIR \
  --save_checkpoint=true \
  --batch_size=$BATCH_SIZE \
  --steps=$NUM_STEPS \
  --save_freq=$SAVE_FREQ \
  --log_freq=$LOG_FREQ \
  --policy.push_to_hub=true \
  --policy.type=groot \
  --policy.repo_id=$REPO_ID \
  --policy.tune_diffusion_model=false \
  --dataset.repo_id=$DATASET_ID \
  --wandb.enable=true \
  --wandb.disable_artifact=true \
  --job_name=$JOB_NAME
```
- eval
```
lerobot-record \
  --robot.type=bi_so_follower \
  --robot.left_arm_port=/dev/ttyACM1 \
  --robot.right_arm_port=/dev/ttyACM0 \
  --robot.id=bimanual_follower \
  --robot.cameras='{ right: {"type": "opencv", "index_or_path": 0, "width": 640, "height": 480, "fps": 30}, left: {"type": "opencv", "index_or_path": 2, "width": 640, "height": 480, "fps": 30}, top: {"type": "opencv", "index_or_path": 4, "width": 640, "height": 480, "fps": 30} }' \
  --display_data=true \
  --dataset.repo_id=${HF_USER}/eval_groot_bimanual \
  --dataset.num_episodes=10 \
  --dataset.single_task="Grab and handover the red cube to the other arm" \
  --policy.path=${HF_USER}/groot-bimanual \
  --dataset.episode_time_s=30 \
  --dataset.reset_time_s=10
```

# Acknowledgements
```
https://wiki.seeedstudio.com/cn/lerobot_so100m_new/
```
