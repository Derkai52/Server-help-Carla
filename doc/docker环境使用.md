# 服务器 Docker 环境

## 目前提供的基础环境：
镜像名    | 备注 |  环境信息  |
-------- | ----- | ----- |
[example:latest](../DockerfileZoo/example_dockerfile)|自定义镜像模版| cuda11.2 | 
[carla:base](../DockerfileZoo/carla_dockerfile)  | 基础 Carla | Carla-0.9.12 pytorch-1.10(cuda11.2)
[carla:rl](../DockerfileZoo/carla_rl_dockerfile)  | 强化学习  Carla | Carla-0.9.12 pytorch-1.10(cuda11.2) Gym
 [carla:control](../DockerfileZoo/carla_control_dockerfile) | 控制算法 Carla | ROS-Noetic Carla-0.9.12 pytorch-1.10(cuda11.2)
| TDB | TDB | TDB


## 使用示例
[Docker 基础使用示例](./example_docker.md)

[Carla Demo 1: 生成 KITTI 数据集](./carla_data_collect.md)

[Carla Demo 2: 使用 Gym 训练强化学习模型实现环岛驶入驶出](./carla_gym_rl.md)

[Carla Demo 3: 联合 ROS 进行横向&纵向控制](./carla_ros_control.md)

