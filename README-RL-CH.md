# 在 SO101 机械臂上跑通强化学习

> 本文位于 [https://github.com/EmbodiedFX/LeRobot-SO101](https://github.com/EmbodiedFX/LeRobot-SO101)，English version: [README-RL.md](README-RL.md)

## 一、 背景和适用环境

本文默认读者已完整跑通《[在 SO101 机械臂上跑通数据采集、模型微调和真机推理](https://github.com/EmbodiedFX/LeRobot-SO101/blob/main/README-CH.md)》的流程。
适用环境也同该教程的描述。

> 本教程主要参照[这个网页](https://huggingface.co/docs/lerobot/hilserl)制作。

## 二、MacBook 和机械臂上的环境准备

在`lerobot`的 conda 环境里：

1. 安装额外需要的 Python 库：
  ```bash
  pip install -e ".[hilserl]"
  ```
2. 在[官方渠道]下载 SO101 机械臂的 URDF 文件，拷贝到`lerobot`项目目录，通过以下方式：
  ```bash
  # 在`lerobot`项目外的一个路径
  git clone --branch main --single-branch git@github.com:TheRobotStudio/SO-ARM100.git
  export SO_PATH=<absolute path to SO-ARM100 root folder>

  # 然后回到 lerobot 项目根目录
  ```
4. 然后，运行如下命令，显示`RECORDING STARTED`后，模拟机械臂完成任务会达到的各种状态，以探测机械臂的活动范围边界：
  ```bash
  # 每用一个新的 Follower 机械臂都要做一次
  lerobot-find-joint-limits \
    --robot.type=so100_follower \
    --robot.port=$FOLLOWER_PORT \
    --robot.id=my_awesome_follower_arm \
    --teleop.type=so100_leader \
    --teleop.port=$LEADER_PORT \
    --teleop.id=my_awesome_leader_arm \
    --urdf_path=./so101_new_calib.urdf
  # urdf_path 参数中的 "./" 不能丢！否则会报错如“Error initializing kinematics: Mesh assets/base_motor_holder_so101_v1.stl could not be found.”

  # 最终的结果类似如下：
  ========================================
  FINAL RESULTS
  ========================================
  
  # End Effector Bounds (x, y, z):
  max_ee = [0.3976, 0.267, 0.3751]
  min_ee = [0.0582, -0.2034, 0.0712]
  
  # Joint Position Limits (radians):
  max_pos = [79.956, 70.2418, 92.7473, 102.1099, 47.6044, 15.8038]
  min_pos = [-78.6374, -103.6484, -94.6813, 24.9231, -6.4615, 1.0218]
  ```
5. 用上面`max_ee`和`min_ee`的数据更新脚本`rl_record_macos.sh`中的参数`env.processor.inverse_kinematics.end_effector_bounds`，如：
  ```bash
  --env.processor.inverse_kinematics.end_effector_bounds="{min: [0.05, -0.20, 0.03], max: [0.39, 0.26, 0.37]}"
  ```

## 三、离线数据集构建

1. 将环境变量`RL_DATA_PATH`设置为目标数据集的路径（这个路径默认相对于`$HF_HOME/lerobot`，其中`$HF_HOME`默认为`$HOME/.cache/huggingface`）：
```bash
export RL_DATA_PATH=local/record-rl
```
2. 执行命令
```bash
chmod +x ./rl_record_macos.sh  # 只需跑一次
./rl_record_macos.sh
```

## Remarks

1. 要看`python -m lerobot.rl.gym_manipulator`接受的参数可以通过`python -m lerobot.rl.gym_manipulator --help`获取指引。同时，主要的参数在[此文档](https://huggingface.co/docs/lerobot/hilserl#understanding-configuration）也有提及。

