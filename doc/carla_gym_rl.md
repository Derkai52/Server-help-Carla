# Carla 使用 Gym 训练实现环岛驶入驶出

项目地址：[https://github.com/cjy1992/gym-carla.git](https://github.com/cjy1992/gym-carla.git)

#
## 构建容器（若已有则直接下一步）
```shell
# 构建镜像
cd DockerfileZoo
docker build -f carla_rl_dockerfile -t carla:rl .
# 创建容器，启用显卡，并将宿主机 docker_ws 目录映射到容器内
docker run --privileged --gpus all --network=host -e DISPLAY=$DISPLAY -v /usr/share/vulkan/icd.d:/usr/share/vulkan/icd.d -v ~/docker_ws:/home/docker_ws --name carla_rl -it carla:rl /bin/bash
```

## 启动示例容器
```bash
# 开启终端一，启动容器的 Carla 服务器
docker start carla_rl
docker exec -it carla_rl /bin/bash ./carla/CarlaUE4.sh

# 开启终端二，进入工具目录
docker exec -it carla_rl /bin/bash
cd gym-carla-master/
```

## 开始训练
```bash
python3 test.py
```