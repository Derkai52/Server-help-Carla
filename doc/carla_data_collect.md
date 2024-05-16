# Carla 数据集采集示例

## 文档参考
项目地址：[https://github.com/Derkai52/CarlaFLCAV.git](https://github.com/Derkai52/CarlaFLCAV.git)

Carla 仿真平台文档: https://carla.readthedocs.io/en/0.9.12/

KITTI-3D 数据集: https://www.cvlibs.net/datasets/kitti/eval_object.php?obj_benchmark=3d

#
## 构建容器（若已有则直接下一步）
```shell
# 构建镜像
cd DockerfileZoo
docker build -f carla_datacollect_dockerfile -t carla:base .
# 创建容器，启用显卡，并将宿主机 docker_ws 目录映射到容器内
docker run --privileged --gpus all --network=host -e DISPLAY=$DISPLAY -v /usr/share/vulkan/icd.d:/usr/share/vulkan/icd.d -v ~/docker_ws:/home/docker_ws --name carla_data_collect -it carla:base /bin/bash
```


## 启动示例容器
```bash
# 开启终端一，启动容器的 Carla 服务器
docker start carla_data_collect
docker exec -it carla_data_collect /bin/bash ./carla/CarlaUE4.sh

# 开启终端二，进入工具目录
docker exec -it carla_data_collect /bin/bash
cd CarlaFLCAV/FLDatasetTool/
```
## 数据集构建
### 1、数据采集
```bash
python3 data_recorder.py
```
### 2、数据标注
以 KITTI 数据集格式标注指定文件夹，名称以实际为准
```
python3 label_tools/kitti_objects_label.py -r record_2023_0910_2303
```

### 3、数据可视化（可从 lidar、semantic lidar 和 radar 中选择）
对于 npy 点云原始数据，使用 util/visualize_lidar.py 可视化
```bash
# 查看单个文件
python3 utils/visualize_lidar.py --type [lidar / semantic_lidar / radar] --source raw_data/record_[date]/[uid]_[vehicle_type]/[uid]_[sensor_type].npy
# 连续播放
python3 utils/visualize_lidar.py --type [lidar / semantic_lidar / radar] --source raw_data/record_[date]/[uid]_[vehicle_type]/[uid]_[sensor_type]
```

#
## 常用采集参数设置
## 一、交通流设置
场景中交通流运行规则详见： [Carla交通流系统](https://carla.readthedocs.io/en/0.9.12/adv_traffic_manager/)
* 修改配置文件 [actor_settings_template.json](../FLDatasetTool/config/actor_settings_template.json) 以修改交通流生成信息。详见 [Carla机动车生成点](https://carla.readthedocs.io/en/0.9.12/core_actors/#spawning)
```bash
"actors": # 关注车：采集各种传感器数据（RGB、雷达、车辆状态等）
[
    {
        "type": "vehicle.tesla.model3",                   # 车辆类型（Carla内置）
        "name": "vehicle.tesla.model3.master",            # 车辆名称
        "sensors_setting": "sensor_config_template.json", # 使用的传感器配置文件（详见下文）
        "spawn_point": 73                                 # 生成点序列号（使用道路预设好的生成点）
    }, 
    {
        "type": "vehicle.tesla.model3",                   
        "name": "vehicle.tesla.model3.second",
        "sensors_setting": "sensor_config_template.json",
        "spawn_point": 78
    },

    ......  # 可以此为模版增添任意规模的交通体
    
],
"other_vehicles": # 其他车：仅采集车辆状态信息
{
    "vehicle_num": 50, # 其他车规模大小
    "spawn_points": [44, 55, 64, 39, 58, 10, 95, 3, 13, 100, 5, 26, 25] # 生成点序列号（使用道路预设好的生成点）
}
```

## 二、天气设置
天气系统提供了天气自动变换和静态天气两种模式。天气系统仅影响 RGB 相机，且各参数的影响效果独立互不干扰（以便于参数控制）。详见 [Carla天气系统](https://carla.readthedocs.io/en/0.9.12/python_api/#carlaweatherparameters)


* 修改配置文件 [world_config_template.json](../FLDatasetTool/config/world_config_template.json) 以修改天气系统参数
~~~ bash
"weather":
{
    "dynamic_weatherd": 1        # 当值为 1 时，表示开启自动天气变换，下面参数将不作用。当值为 0 时，使用下面参数生成静态天气
    "sun_altitude_angle": 15,    # 90 是中午，-90 是午夜
    "cloudiness": 0,             # 0是干净的天空，100是厚重的云
    "precipitation": 0,          # 雨水，0是不下雨，100是最大的雨
    "precipitation_deposits": 0, # 水坑的创建。值范围从 0 到 100，即 0 完全没有，100 表示完全被水覆盖的道路。
    "wind_intensity": 0,         # 它会影响雨水
    "fog_density": 0,            # 雾厚度，最大为100
    "fog_distance": 0,           # 雾可见距离。值范围从 0 到无限。
    "fog_falloff": 0,            # 雾的密度，从0到无穷大。值越大，密度越大，雾气会到达更小的高度
}
~~~

## 三、切换场景地图
Carla 提供了数个规模与风格不一的场景地图，这里推荐使用 `Town02`, `Town05`, `Town10HD` 以保证存在较高密度的交通流，提高数据采集效率。

* 修改配置文件 [world_config_template.json](../FLDatasetTool/config/world_config_template.json) 以切换场景地图 
```bash
# 可以使用的地图有： Town01、Town02、Town03、Town04、Town05、Town10HD
{
    "map": "Town02"
}
```

**Note:** 本文档示例默认为 `Town02` 定制世界生成配置文件，因此在更换其他场景时，建议使用新的参数 (例如路侧传感器位姿、车辆生成点信息等)

## 四、车辆速度限制
* 修改配置文件 [world_config_template.json](../FLDatasetTool/config/world_config_template.json) 以修改车辆速度上限 
~~~bash
# 设置车辆的预期速度和当前速度限制的百分比差异。设置为负值可以超出速度限制。预期车速最高默认值为 30km/h。
    "speed_limit":
    {
        "global_percentage_speed_difference": 0.0   # float类型 0.0 为默认速度， 100.0 为车辆静止， -100.0 为车速翻倍  
    }
~~~


## 五、传感器设置
由于使用 Carla 提供的车型与传感器布局，最终数据采集结果与 KITTI 等主流数据集采集平台存在一些偏差，按需调整即可。详见 [Carla传感器系统](https://carla.readthedocs.io/en/0.9.12/ref_sensors/)

* 修改配置文件 [sensor_config_infrastructure_template.json](../FLDatasetTool/config/sensor_config_infrastructure_template.json) 以修改路侧传感器参数

* 修改配置文件 [sensor_config_template.json](../FLDatasetTool/config/sensor_config_template.json) 以修改车载传感器参数

```bash
"sensors":
{
    "type": "sensor.camera.rgb",   # RGB相机传感器
    "name": "image_2",
    "spawn_point": {"x": 2.0, "y": 0.0, "z": 2.0, "roll": 0.0, "pitch": 0.0, "yaw": 0.0},         # 传感器相对车中心安装位姿
    "image_size_x": 1382,          # 采集分辨率
    "image_size_y": 512,
    "fov": 90.0                    # 视场角
},
{
    "type": "sensor.lidar.ray_cast", # 雷达传感器
    "name": "velodyne_64",
    "spawn_point": {"x": 0.0, "y": 0.0, "z": 2.4, "roll": 0.0, "pitch": 0.0, "yaw": 0.0},         # 传感器相对车中心安装位姿
    "range": 100,
    "channels": 64,                # 线数
    "points_per_second": 1300000,  # 每秒点云量
    "upper_fov": 2.0,              # 视场角
    "lower_fov": -24.8,
    "rotation_frequency": 10,      # 检测频率
    "noise_stddev": 0.02           # 噪声方差
}
```