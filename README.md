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
conda create -n env_lerobot python=3.10 && conda activate lerobot
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
