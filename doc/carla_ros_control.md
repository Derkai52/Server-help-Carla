# Carla 联合 ROS 纵向&横向控制算法

项目地址：[https://github.com/Derkai52/Carla-Ros-Bridge-PNC.git](https://github.com/Derkai52/Carla-Ros-Bridge-PNC.git)

#
## 构建容器（若已有则直接下一步）
```shell
# 构建镜像
cd DockerfileZoo
docker build -f carla_control_dockerfile -t carla:control .
# 创建容器，启用显卡，并将宿主机 docker_ws 目录映射到容器内
docker run --privileged --gpus all --network=host -e DISPLAY=$DISPLAY -v /usr/share/vulkan/icd.d:/usr/share/vulkan/icd.d -v ~/docker_ws:/home/docker_ws --name carla_control -it carla:control /bin/bash
```

## 一、控制算法 Demo
### 开启终端一（启动 Carla 服务器）
```bash
docker start carla_control
docker exec -it carla_control /bin/bash ./carla/CarlaUE4.sh
```

### 开启终端二 (启动控制节点)
```shell
docker exec -it carla_control /bin/bash

cd ./Carla-Ros-Bridge-PNC/catkin_ws

source devel/setup.bash

# Pure Pursuit
roslaunch controller controller_demo.launch control_method:="PurePursuit"

# Stanley
roslaunch controller controller_demo.launch control_method:="Stanley"

# LQR
roslaunch controller controller_demo.launch control_method:="LQR"
```

## 二、规划算法 Demo
### 开启终端一（启动 Carla 服务器）
```bash
docker start carla_control
docker exec -it carla_control /bin/bash ./carla/CarlaUE4.sh
```

### 开启终端二（启动规划节点）
```shell
docker exec -it carla_control /bin/bash

cd ./Carla-Ros-Bridge-PNC/catkin_ws

source devel/setup.bash

roslaunch planning planning_demo.launch
```

### 开启终端三（启动控制节点）
```shell
docker exec -it carla_control /bin/bash

cd ./Carla-Ros-Bridge-PNC/catkin_ws

source devel/setup.bash

roslaunch controller controller.launch
```
