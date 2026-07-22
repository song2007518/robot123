# GENISOM-AI RoamerX Open 配置说明

## 环境要求

| 组件 | 版本/说明 |
|---|---|
| 操作系统 | Ubuntu 22.04 LTS (Jammy) |
| ROS2 | Humble Hawksbill |
| 仿真平台 | MATRiX v0.1.2 |
| 中间件 | rmw_zenoh_cpp |
| 机器人型号 | xgw（轮式四足） |
| 可用场景 | 仅 Yard（场景 3） |

## 目录结构

```
~/robot_ws/
├── genisom_roamerx_open/   # 导航栈（本仓库）
├── matrix/                 # MATRiX 仿真平台
├── genisom_model/          # 机器人 URDF 模型
├── genisom_l1_sdk_old/     # L1 SDK（git clone 获取）
└── genisom_robot_sdk/      # 机器人 SDK（参考用）
```

## 安装步骤

### 1. 克隆仓库

```bash
mkdir -p ~/robot_ws && cd ~/robot_ws
git clone https://github.com/zsibot/genisom_roamerx_open.git
git clone https://github.com/zsibot/matrix.git
git clone https://github.com/zsibot/genisom_model.git
git clone https://github.com/zsibot/genisom_l1_sdk_old.git
```

### 2. 安装 MATRiX 依赖和地图

```bash
cd ~/robot_ws/matrix
bash scripts/install_deps.sh
bash scripts/release_manager/install_chunks.sh 0.1.2
```

### 3. 安装 ROS2 依赖

```bash
sudo apt install ros-humble-slam-toolbox ros-humble-pointcloud-to-laserscan ros-humble-nav2-amcl
```

### 4. 编译导航栈

```bash
cd ~/robot_ws/genisom_roamerx_open
./build.sh all
```

## 关键配置

### MC 配置

- `matrix/src/robot_mc/build/export/config/robot-defaults.yaml`：`use_gamepad: 0`
- `matrix/src/robot_mc/build/export/config/sdk_config.yaml`：`target_port: 43988`

### 环境变量（~/.bashrc）

```bash
source /opt/ros/humble/setup.bash
export ROS_DOMAIN_ID=89
export RMW_IMPLEMENTATION=rmw_zenoh_cpp
export SDK_CLIENT_IP=127.0.0.1
source ~/robot_ws/genisom_roamerx_open/install/setup.bash
```

## 使用方式

### 启动仿真和导航

```bash
cd ~/robot_ws/matrix
./bin/sim_launcher
# 选择 xgw 机器人 + Yard 场景
# 勾选 MuJoCo physics
# 点击开始 — 等 UE/MC/MuJoCo 完全加载
# 再勾选 RoamerX Open — 启动导航
# ⚠️ 顺序不能错：必须先开始仿真，再开导航，否则机器人站不起来
```

### RViz 操作

1. RViz 弹出后，用 **"2D Pose Estimate"** 在地图上对应位置设初始位姿
2. 用 **"2D Goal Pose"** 发导航目标
3. 机器人将自主规划路径并导航

## 重要说明

1. **只能用 Yard（场景 3）**：导航的 `map.yaml` 是为 Yard 场景预建的，其他场景地图不匹配
2. **必须勾选 MuJoCo physics**：否则 MC 不产生电机输出
3. **每次只有第一次冷启动正常**：sim_launcher 反复开关可能导致 sensor_sim 节点丢失，需完全退出重启
4. **`sdk_bridge.py` 是机器人控制核心**：通过 L1 SDK 直连 MC，替代了原始的 `vel_cmd_udp_pub`

## 已修改的核心参数

| 参数 | 位置 | 值 | 说明 |
|---|---|---|---|
| use_gamepad | robot-defaults.yaml | 0 | 无需手柄控制 |
| yaw_goal_tolerance | navigo_params.yaml | 1.0 | 到达后不转圈 |
| observation_persistence | navigo_params.yaml | 1.0 | 代价图不闪烁 |
| SLAM lid_topic | robot_slam config | /livox/lidar | 匹配仿真点云 |

## 已知问题

1. **slam_toolbox 无法用于导航**：`global_costmap/static_layer` 的 QoS 与 slam_toolbox 不兼容，测试结果为不可行
2. **`rmw_zenoh_cpp` 不稳定**：可能导致 ROS2 Service 超时、CLI 命令挂死，需重启计算机恢复
3. **sim_launcher 多次重启后传感器节点丢失**：只有完全退出重开才能恢复
# robot-2nd
