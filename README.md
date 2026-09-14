# AirportTrolley

机场行李手推车机器人相关代码集合，包含移动底盘、双机械臂、CAN 电机驱动、激光定位与建图、视觉伺服、推车控制，以及避障/规划仿真等模块。

<p align="center">
  <img src="assets/airport_trolley_demo.gif" alt="Airport trolley robot demonstration" width="640">
</p>

## 相关论文

1. **DA-VPC: Disturbance-Aware Visual Predictive Control Scheme of Docking Maneuvers for Autonomous Trolley Collection**  
   Yuhan Pang, Bingyi Xia, Zhe Zhang, Zhirui Sun, Peijia Xie, Bike Zhu, Wenjun Xu, and Jiankun Wang. arXiv:2509.07413, 2025.  
   [[Paper](https://arxiv.org/pdf/2509.07413)] · 相关模块：`ibvs_ws`

2. **Integrating Maneuverable Planning and Adaptive Control for Robot Cart-Pushing under Disturbances**  
   Zhe Zhang, Peijia Xie, Yuhan Pang, Zhirui Sun, Bingyi Xia, Bi-Ke Zhu, and Jiankun Wang. arXiv:2506.18410, 2025.  
   [[Paper](https://arxiv.org/pdf/2506.18410)] · 相关模块：`push_ws`、`ibvs_arm_ws`

3. **Collaborative Trolley Transportation System with Autonomous Nonholonomic Robots**  
   Bingyi Xia, Hao Luan, Ziqi Zhao, Xuheng Gao, Peijia Xie, Anxing Xiao, Jiankun Wang, and Max Q.-H. Meng. *2023 IEEE/RSJ International Conference on Intelligent Robots and Systems (IROS)*, pp. 8046–8053, 2023.  
   [[Paper](https://ieeexplore.ieee.org/stamp/stamp.jsp?arnumber=10341508)] [[DOI](https://doi.org/10.1109/IROS55552.2023.10341508)] · 相关模块：`airport_ws`

4. **Autonomous Multiple-Trolley Collection System with Nonholonomic Robots: Design, Control, and Implementation**  
   Peijia Xie, Bingyi Xia, Anjun Hu, Ziqi Zhao, Lingxiao Meng, Zhirui Sun, Xuheng Gao, Jiankun Wang, and Max Q.-H. Meng. *Journal of Field Robotics*, 42(1):20–36, 2025.  
   [[Paper](https://onlinelibrary.wiley.com/doi/full/10.1002/rob.22395)] [[DOI](https://doi.org/10.1002/rob.22395)] · 相关模块：`airport_ws`

本仓库是一个聚合仓库（superproject）：下列 6 个目录都是独立 Git 子模块，各自对应一个可单独构建的 ROS 2 工作空间源码仓库。

## 仓库结构

| 子模块 | 主要作用 | 典型场景 |
| --- | --- | --- |
| [`airport_ws`](airport_ws) | 机场车完整系统：底盘驱动、建图定位、路径规划、推车识别/接近、夹爪与任务状态机 | 机场场景实机集成 |
| [`can_ws`](can_ws) | USB-CAN、DM4310 电机、双臂协同、力传感和 NMPC 底层控制 | 执行器与传感器驱动 |
| [`ibvs_arm_ws`](ibvs_arm_ws) | 双机械臂运动学、逆运动学和关节控制 | 机械臂独立调试 |
| [`ibvs_ws`](ibvs_ws) | 基于 MPC 的图像视觉伺服、UWB 数据解析、自定义消息与动作接口 | 视觉对准和抗遮挡实验 |
| [`push_ws`](push_ws) | 双臂推/拉行李车、力控、推车运动学及障碍物处理 | 人机协作推车实验 |
| [`pyh_ws`](pyh_ws) | AO/CBF/MPC 规划、仿真封装、VRPN 定位和算法对比 | 规划算法仿真与实机验证 |

> [!IMPORTANT]
> 不要直接在本仓库根目录运行 `colcon build`。`airport_ws` 与 `push_ws` 中存在若干同名 ROS 包，必须进入具体子模块分别构建。

## 环境

当前代码使用的开发环境为：

- Ubuntu 20.04
- ROS 2 Galactic
- Python 3、C++17
- `colcon`、`rosdep`

部分模块还需要按实际硬件和算法配置安装：

- CasADi（部分控制器固定链接 `/usr/local/lib/libcasadi.so.3.7`）
- SNOPT/IPOPT（`push_ws` 中的优化控制可选）
- RealSense SDK、PCL、Eigen、OctoMap
- Ouster/激光雷达驱动、VRPN、USB-CAN 驱动
- DM4310 电机、力传感器、夹爪等对应硬件

## 获取代码

首次克隆时同时拉取全部子模块：

```bash
git clone --recurse-submodules https://github.com/yh83305/AirportTrolley.git
cd AirportTrolley
```

如果已经克隆了主仓库：

```bash
git submodule update --init --recursive
```

更新主仓库和所有子模块：

```bash
git pull
git submodule update --init --recursive
```

## 构建方法

每个子模块都保存了原工作空间的 `src/` 内容，但子模块根目录本身也可以作为工作空间根目录使用。例如：

```bash
source /opt/ros/galactic/setup.bash
cd AirportTrolley/airport_ws

rosdep install --from-paths . --ignore-src -r -y
colcon build --symlink-install
source install/setup.bash
```

其他子模块同理：

```bash
cd ../can_ws
rosdep install --from-paths . --ignore-src -r -y
colcon build --symlink-install
source install/setup.bash
```

建议每个终端都重新执行相应的 `source`。需要跨工作空间运行时，先加载底层工作空间，再加载应用工作空间，例如：

```bash
source /opt/ros/galactic/setup.bash
source /path/to/AirportTrolley/can_ws/install/setup.bash
source /path/to/AirportTrolley/push_ws/install/setup.bash
```

## 模块说明与基本启动

以下命令以各子模块已经成功构建为前提。硬件节点应分终端启动，并在上电前确认设备名、CAN ID、关节零位、限位和急停状态。

### 1. `airport_ws`：机场车系统集成

主要包：

- `air_robot`：移动底盘、遥控、直线电机和夹爪控制；主要节点为 `airCar`、`joy`、`linear_motor_controller`、`gripper_controller`。
- `FAST_LIO`：激光雷达-惯性里程计和建图。
- `fast_lio_localization`：加载点云地图并进行定位，包含地图服务。
- `lf_path_planner`：Hybrid A* / A* 路径规划及服务节点。
- `local_planner`：基于 NMPC 的局部接近控制。
- `whole_planner`：机场车任务状态机，串联接近、对接和导航流程。
- `trolley_collection`：手推车任务控制与抓取实验入口。
- `find_plane`：基于点云的平面检测。
- `vrpn_listener`：动捕数据接收。
- `octomap_server`、`map_srv_interface`：三维地图和地图服务接口。
- `fashion_gripper_controller1`：Fashion Star 舵机夹爪控制。
- `realsense2_camera*`、`cmd_vel_mux`：RealSense 相机和速度指令复用。

常用启动示例：

```bash
cd /path/to/AirportTrolley/airport_ws
source install/setup.bash

# FAST-LIO 建图
ros2 launch fast_lio mapping.launch.py

# 已有地图定位（启动前检查配置中的地图路径）
ros2 launch fast_lio_localization fast_lio_localizaton.launch.py

# 地图加载/服务
ros2 launch fast_lio_localization map.launch.py

# 任务控制与规划
ros2 run trolley_collection controller
ros2 run whole_planner whole_fsm

# 平面检测
ros2 run find_plane find_plane
```

底盘和末端执行器可按需分别启动：

```bash
ros2 run air_robot airCar
ros2 run air_robot gripper_controller
ros2 run fashion_gripper_controller1 fashion_servo
```

### 2. `can_ws`：CAN、电机与力传感

主要包：

- `usbcan`：USB-CAN 通信，节点 `can_node`。
- `dm4310_controller`：DM4310 电机反馈与控制，节点 `dm4310_node`。
- `arm_force`：机械臂力传感数据读取，节点 `force_node`。
- `arm_cooperate`：双机械臂协同控制，节点 `dual_arm_controller`。
- `airport_controller`：机场车 NMPC 底盘控制，节点 `nmpc_ctrl`。

基本启动顺序：

```bash
cd /path/to/AirportTrolley/can_ws
source install/setup.bash

# 终端 1：CAN 通信
ros2 run usbcan can_node

# 终端 2：电机控制
ros2 run dm4310_controller dm4310_node

# 终端 3：按实验需要启动
ros2 run arm_force force_node
ros2 run arm_cooperate dual_arm_controller
ros2 run airport_controller nmpc_ctrl
```

### 3. `ibvs_arm_ws`：双机械臂控制

`arms_control` 提供双臂正/逆运动学、Jacobian、关节 PD、力矩补偿、速度估计和力矩变化率限制。机械臂状态通常来自 `dm_motor_feedback`，控制指令发送到 `dm_motor_control`。

先启动 `can_ws` 中的 CAN 与电机节点，再运行：

```bash
cd /path/to/AirportTrolley/ibvs_arm_ws
source /path/to/AirportTrolley/can_ws/install/setup.bash
source install/setup.bash

ros2 run arms_control test_dual_arm
```

调试前应先打印并核对左右臂关节角。不要在零位、方向和限位未确认时直接下发控制量。

### 4. `ibvs_ws`：MPC 图像视觉伺服

主要包：

- `mpc_ibvs`：图像特征检测、位姿/偏航估计、MPC 控制、动作服务和实验数据处理。
- `custom_msgs`：视觉伺服使用的自定义消息/动作接口。
- `nlink_parser_ros2`：Nooploop LinkTrack/UWB 数据解析。
- `nlink_parser_ros2_interfaces`：UWB 消息定义。

UWB 定位：

```bash
cd /path/to/AirportTrolley/ibvs_ws
source install/setup.bash

ros2 launch nlink_parser_ros2 linktrack.launch.py
ros2 topic echo /nlink_linktrack_nodeframe2
```

视觉伺服基础节点：

```bash
# 图像检测与偏航估计
ros2 run mpc_ibvs ir_mpc_detect_yaw_IPPE

# 根据实验选择一个动作服务
ros2 run mpc_ibvs action_server_nominal
ros2 run mpc_ibvs action_server_compensate
ros2 run mpc_ibvs action_server_estiminate
ros2 run mpc_ibvs action_server_occlusion

# 另一个终端发送动作目标
ros2 run mpc_ibvs action_client 1
```

运行前需要确认相机话题、相机标定、目标特征、动作参数以及底盘速度话题与实际系统一致。

### 5. `push_ws`：双臂推车与力控制

主要包：

- `trolley_kinematics`：推车运动学、双臂推/拉状态机和扇区障碍物处理；节点 `push_trolley`、`sector_obstacle_node`。
- `arms_control_zz`：机械臂控制 Python/C++ 扩展。
- `trolley_description`、`fishbot_description`：手推车和移动机器人模型/仿真描述。
- `air_robot`：底盘和夹爪控制。
- `lf_path_planner`、`local_planner`、`whole_planner`：路径规划、局部接近和任务状态机。
- `FAST_LIO`、`fast_lio_localization`、`octomap_server`、`find_plane`：感知、建图、定位和障碍处理。

实机基本启动顺序：

```bash
# 每个命令建议使用独立终端，并加载对应工作空间

# can_ws：CAN、电机和力传感
source /path/to/AirportTrolley/can_ws/install/setup.bash
ros2 run usbcan can_node
ros2 run dm4310_controller dm4310_node
ros2 run arm_force force_node

# push_ws：夹爪、障碍处理和推车主节点
source /path/to/AirportTrolley/push_ws/install/setup.bash
ros2 run air_robot gripper_controller
ros2 run trolley_kinematics sector_obstacle_node
ros2 run trolley_kinematics push_trolley
```

`push_trolley` 包含 Push、Soft、Turn、Draw 和 Stay 等控制状态。使用 CasADi/SNOPT/IPOPT 时，需要确保动态库路径已配置，例如：

```bash
export LD_LIBRARY_PATH="$HOME/local/lib:$HOME/local/lib/casadi:$LD_LIBRARY_PATH"
export CASADIPATH="$HOME/local/lib:$HOME/local/lib/casadi"
```

### 6. `pyh_ws`：AO、CBF/MPC 规划与仿真

主要包：

- `robot_sim_wrapper`：机器人和环境仿真封装。
- `ao`：AO 仿真算法代码。
- `ao_real`：AO 实机运行与 VRPN 相对位姿处理。
- `localization`：多实体定位、速度/加速度估计和数据中心。
- `vrpn_tf_broadcaster`：VRPN 位姿到 TF 的广播。
- `mpc_planner`：障碍物检测、Hybrid A*、CBF/MPC/CVaR 规划与轨迹跟踪。
- `comparison`：定位或算法结果对比。

仿真启动：

```bash
cd /path/to/AirportTrolley/pyh_ws
source install/setup.bash

ros2 launch robot_sim_wrapper robot_simulation.launch.py
```

VRPN/实机定位链路：

```bash
ros2 launch robot_sim_wrapper env.launch.py
ros2 launch vrpn_tf_broadcaster vrpn.launch.py
ros2 launch localization localization.launch.py
ros2 run ao_real vrpn_relative_node
```

CBF/MPC 规划常用节点：

```bash
# 障碍物检测
ros2 run mpc_planner bf_detector

# Hybrid A* 全局规划
ros2 run mpc_planner global_planner_ha
ros2 run mpc_planner a_plan

# 规划与轨迹跟踪（按实验选择）
ros2 run mpc_planner cbf_planner
ros2 run mpc_planner cvar_planner
ros2 run mpc_planner simple_planner
ros2 run mpc_planner cbf_tracker
```

## 常用检查命令

```bash
# 查看节点和话题
ros2 node list
ros2 topic list

# 查看 TF 树
ros2 run tf2_tools view_frames

# 查看某个包是否已正确安装
ros2 pkg prefix <package_name>

# 只构建指定包
colcon build --symlink-install --packages-select <package_name>
```

## 数据与大文件说明

本项目公开仓库以源码为主，上传时未包含：

- `build/`、`install/`、`log/`
- 嵌套仓库的 `.git/`
- 点云地图 `*.pcd`
- 演示 GIF、已编译的 `*.so`

因此使用 `fast_lio_localization` 前，需要自行准备地图文件并修改对应配置路径。部分机器人 DAE/STL mesh 属于运行所需模型资源，已保留在仓库中。

## 安全提示

- 启动电机、机械臂、夹爪或底盘前，确认急停可用并清空运动范围。
- 先核对 CAN 接口、设备权限、电机 ID、关节方向、零位和限位，再加载控制器。
- 初次测试应降低速度、力矩和刚度参数，并持续观察反馈话题。
- 不要把访问令牌、密码、设备密钥或包含敏感数据的 rosbag 提交到公开仓库。

