[上一页：比赛介绍](README.md)

---

## 2.1. 系统安装
### 2.1.1 获取比赛安装包
- 下载 [**`Carla比赛安装包`**](https://carla-releases.s3.eu-west-3.amazonaws.com/Linux/Leaderboard/CARLA_Leaderboard_20.tar.gz)。

> 此版本的 Carla 基于 0.9.13 做了修改，包含了一些 0.9.14 的新特性，最新 [CARLA 文档](https://carla.readthedocs.io/en/latest/) 的 API 可作为参考。

- 将安装包解压缩到一个文件夹中, 如 `~/workdir`
```bash
export WORK_DIR=~/workdir
mkdir -p ${WORK_DIR}
cd ${WORK_DIR}
tar -zcvf CARLA_Leaderboard_20.tar.gz carla
```
- 设置 **`${CARLA_ROOT}`** 变量为`Carla`根目录
```bash
export CARLA_ROOT=${WORK_DIR}/carla
```
 
- 为了使用 Carla Python API，需要安装一些Python依赖, 您可以使用您喜欢的 Python 环境，例如对于conda：
```bash
conda create -n py37 python=3.7
conda activate py37
cd ${CARLA_ROOT}  # Change ${CARLA_ROOT} for your CARLA root folder
pip3 install -r PythonAPI/carla/requirements.txt
```

### 2.1.2 安装 leaderboard 和 Scenario_Runner
- 下载大赛 [leaderboard](https://github.com/carla-simulator/scenario_runner.git) **leaderboard-2.0** 分支
```bash
cd ${WORK_DIR}
git clone -b leaderboard-2.0 --single-branch https://github.com/carla-simulator/leaderboard.git
```

!>  <span style="color: red; font-weight: 1000; text-underline-position: below; text-decoration: underline;"> NEED TO CONFIRM: 我们直接使用的 leaderboard-2.0，里面的 README 会跳转到 leaderboard 官方英文网站，这需要如何处理？</span> 

- 设置 **`${LEADERBOARD_ROOT}`** 变量到您的比赛文件根目录
```bash
export LEADERBOARD_ROOT=${WORK_DIR}/leaderboard
```
- 安装所需的 Python 依赖项
```bash
cd ${LEADERBOARD_ROOT} # Change ${LEADERBOARD_ROOT} for your Leaderboard root folder
pip3 install -r requirements.txt
```
 
- 下载 [Scenario Runner](https://github.com/carla-simulator/scenario_runner.git) 仓库 **leaderboard-2.0** 分支
```bash
cd ${WORK_DIR}
git clone -b leaderboard-2.0 --single-branch https://github.com/carla-simulator/scenario_runner.git
```
 
- 设置 **`${SCENARIO_RUNNER_ROOT}`** 为相应的 Scenario_Runner 根目录
```bash
export SCENARIO_RUNNER_ROOT=${WORK_DIR}/scenario_runner 
```
 
- 使用相同的 Python 环境安装所需的 Python 依赖项
```bash
cd ${SCENARIO_RUNNER_ROOT} # Change ${SCENARIO_RUNNER_ROOT} for your Scenario_Runner root folder
pip3 install -r requirements.txt
```

#### 2.1.2.1 基于ROS环境

如果您需要使用ROS开发环境，请首先下载合适的 ROS 或者 ROS2 环境版本，目前支持 ROS Melodic, ROS Noetic 和 ROS Foxy. 然后构建基于 [**ROS**](https://carla.readthedocs.io/projects/ros-bridge/en/latest/ros_installation_ros1/) 或者 [**ROS2**](https://carla.readthedocs.io/projects/ros-bridge/en/latest/ros_installation_ros2/) 的 `CARLA ROS bridge`, 其代码地址如下：

```bash
git clone --recurse-submodules -b leaderboard-2.0 --single-branch https://github.com/carla-simulator/ros-bridge

```

### 2.1.3 定义环境变量

- 用以下命令打开`~/.bashrc`配置文件
```bash
gedit ~/.bashrc
```

- 编辑您的`~/.bashrc`配置文件，添加下面的定义。编辑后保存并关闭该文件
```bash
export CARLA_ROOT=PATH_TO_CARLA_ROOT
export SCENARIO_RUNNER_ROOT=PATH_TO_SCENARIO_RUNNER
export LEADERBOARD_ROOT=PATH_TO_LEADERBOARD
export PYTHONPATH="${CARLA_ROOT}/PythonAPI/carla/":"${SCENARIO_RUNNER_ROOT}":"${LEADERBOARD_ROOT}":"${CARLA_ROOT}/PythonAPI/carla/dist/carla-0.9.13-py3.7-linux-x86_64.egg":${PYTHONPATH}
```

- 使用以下命令使`.bashrc`设置生效
```bash
source ~/.bashrc
```

## 2.2. 创建 Autonomous Agent
`Autonomous Agent` 可表示为您的自动驾驶算法执行器，比赛系统将会在不同的交通情况下，让您的算法执行器在多条规定的路线下运行，并评估车辆的行为，以下说明会让您对此有更具体的理解：
 
- 打开一个终端，运行 CARLA 服务器
```bash
cd ${CARLA_ROOT}
./CarlaUE4.sh -quality-level=Epic -world-port=2000 -resx=800 -resy=600
```

- 使用另一个终端，进入到 `${LEADERBOARD_ROOT}`，并创建一个 bash 脚本
```bash
cd ${LEADERBOARD_ROOT}
touch test_run.sh
chmod +x test_run.sh
```

- 将以下代码粘贴到 `test_run.sh` 中，这些代码将会设置环境变量，这些环境变量将会传给 `run_evaluation.py`使用
```bash
export ROUTES=${LEADERBOARD_ROOT}/data/routes_devtest.xml #预定义路线
export REPETITIONS=1
export DEBUG_CHALLENGE=1
export TEAM_AGENT=${LEADERBOARD_ROOT}/leaderboard/autoagents/human_agent.py
export CHECKPOINT_ENDPOINT=${LEADERBOARD_ROOT}/results.json
export CHALLENGE_TRACK_CODENAME=MAP
./scripts/run_evaluation.sh
```

- 运行脚本
```bash
./test_run.sh
```

这将启动一个 pygame 窗口，让您选择手动控制一个 agent。 沿着彩色航点指示的路线前进，以到达您的目的地。 此脚本将在地图 Town12 加载`routes_devtest.xml`中预定义的两条路线。

![image](images/Town12route.png)

> 沿着指示路线并按照交通规则行驶至目的地。 手动停止当前运行路线，会自动切换到下一条。

在运行测试时，我们设置或用到了一系列参数，这些参数的作用如下：
 
- `ROUTES (XML)` — 运行路线集合。每条路线都有一个起点（第一个航点），和一个终点（最后一个航点）。此外，它们还可以包含一个天气概况，以设置特定的天气条件。一个`XML`包含多条路线，每一条都有一个ID。 用户可以对路线进行的修改、添加和删除。 我们提供了一套用于调试、训练和验证的路线。 用于在线评估的验证路线是未开放的。 这个文件还包括将在模拟环境中测试的场景，每条路线都有自己的一套场景。一个场景被定义为一个交通场景。 您的`Agent` 必须有效的处理这些交通场景，才能通过测试。 参与者可以获得一套在公开的地图上适用的交通场景。 这些场景列表可以在 [**这里查看**](scenarios.md)。
 
- `REPETITIONS (int)` — 每条路线被重复执行的次数。

- `TEAM_AGENT` (Python 模块) — 您的算法执行的 python 代码入口，是一个路径参数，评估系统会通过此参数调用您的算法。
 
其他相关参数的描述如下：
- `TEAM_CONFIG`（由用户定义）— 您的Agent的配置文件路径。 您需要在 agent 类中解析这个文件。
- `DEBUG_CHALLENGE (int)` — 表示在模拟过程中是否应该显示调试信息的标志。 默认情况下，这个变量是未设置的（0），不会显示任何调试信息。当它被设置为 1 时，模拟器将显示要遵循的参考路线。 如果这个变量被设置为大于1，系统将打印出模拟的完整状态，用于调试。
- `CHECKPOINT_ENDPOINT (JSON)` — 将记录比赛指标的文件名。
- `RECORD_PATH (string)` — 存储CARLA日志的文件夹的路径。默认情况下未设置。
- `RESUME` — 表示是否应该从最后一条路线恢复模拟的标志。默认情况下未设置。
- `CHALLNGE_TRACK_CODENAME` — 表示此次比赛的赛道，有两个支持选项 `SENSORS` 和 `MAP`.
    `SENSORS` 赛道可以使用多个cameras，1 个 LIDAR，1 个 RADAR, 1 个 GNSS, 1 个 IMU 和 1 个 speedometer. 除此之外，对于 `MAP` 赛道，您还可以获取到 OpenDRIVE 高清地图，如果有必要，您需要解析和处理 OpenDRIVE 地图。

!> 目前对于`SENSORS`赛道仅可以本地实验，但是线上评估只支持 `MAP`赛道.

这些环境变量被传递给`${LEADERBOARD_ROOT}/leaderboard/leaderboard_evaluator.py`，它作为执行算法的入口。可以参考`leaderboard_evaluator.py`，了解更多关于您的 agent 将如何被执行和评估的细节。
 
!> <span style="color: red; font-weight: 1000; text-underline-position: below; text-decoration: underline;">TODO: 待确定LEADERBOARD代码仓库地址之后，添加相关文件链接。</span>

## 2.3. 创建您自己的 Autonomous Agent
定义一个新的 agent，首先要创建一个新的类，该类继承于`leaderboard.autoagents.autonomous_agent.AutonomousAgent`. 对于基于 ROS 的 agents，您应当继承自 `leaderboard.autoagents.ros1_agent.ROS1agent` 或 `leaderboard.autoagents.ros2_agent.ROS2agent`.

### 2.3.1 创建 get_entry_point
首先，定义一个名为 get_entry_point 的函数，返回您的新类的名称。这将被用来自动实例化您的 agent。
```python
from leaderboard.autoagents.autonomous_agent import AutonomousAgent

def get_entry_point():
    return 'MyAgent'

class MyAgent(AutonomousAgent):
    ...
```
#### 2.3.1.1 创建基于 ROS 环境的 entry point
对于基于 ROS 的 agent，您也需要定义entry point，在此entry point下，用户可以指定 `package`、`launch_file`和需要的参数。
```bash
from leaderboard.autoagents.ros1_agent import ROS1Agent

def get_entry_point():
    return 'MyROSAgent'


class MyRosAgent(ROS1Agent):
    ....
    def get_ros_entry_point(self):
        return {
            "package": "my_ros_agent",
            "launch_file": "my_ros_agent.launch",
            "parameters": {}
        }

```
### 2.3.2 覆盖 setup 方法
在您的 agent 类中重写 setup 方法。这个方法执行您的 agent 需要的所有初始化和定义。它将在每次路线初始化时被自动调用。 它可以接收一个指向配置文件的可选参数。 用户应该解析这个文件, 另外，您应当指定 `self.track=Track.MAP`.
```python
from leaderboard.autoagents.autonomous_agent import Track
...
def setup(self, path_to_conf_file):
    self.track = Track.MAP # At a minimum, this method sets the Leaderboard modality. In this case, MAP
```
?> `self.track` 属性是枚举类型而非字符串类型. 其取值只能为 `Track.SENSORS` 或 `Track.MAP`。 线上系统目前仅支持 `Track.MAP`。

### 2.3.3 覆盖 sensors 方法
您必须覆盖 sensors 方法，该方法定义了您的 agent 所需的所有传感器。
```python
def sensors(self):
    sensors = [
        {'type': 'sensor.camera.rgb', 'id': 'Center',
         'x': 0.7, 'y': 0.0, 'z': 1.60, 'roll': 0.0, 'pitch': 0.0, 'yaw': 0.0, 'width': 300, 'height': 200, 'fov': 100},
        {'type': 'sensor.lidar.ray_cast', 'id': 'LIDAR',
         'x': 0.7, 'y': -0.4, 'z': 1.60, 'roll': 0.0, 'pitch': 0.0, 'yaw': -45.0},
        {'type': 'sensor.other.radar', 'id': 'RADAR',
         'x': 0.7, 'y': -0.4, 'z': 1.60, 'roll': 0.0, 'pitch': 0.0, 'yaw': -45.0, 'fov': 30},
        {'type': 'sensor.other.gnss', 'id': 'GPS',
         'x': 0.7, 'y': -0.4, 'z': 1.60},
        {'type': 'sensor.other.imu', 'id': 'IMU',
         'x': 0.7, 'y': -0.4, 'z': 1.60, 'roll': 0.0, 'pitch': 0.0, 'yaw': -45.0},
        {'type': 'sensor.opendrive_map', 'id': 'OpenDRIVE', 'reading_frequency': 1},
        {'type': 'sensor.speedometer', 'id': 'Speed'},
    ]
    return sensors
```
 
?> 大多数的传感器属性都有固定的值。 这些可以在`agent_wrapper.py`中查到。这样做是为了让所有参赛队伍都使用同样的传感器框架。
 
每个传感器都被表示为一个字典，包含以下属性：

- `type`：要添加的传感器的类型。

- `id`：将被赋予传感器的标签，以便以后访问。

- `其他属性`：这些属性与传感器有关，例如：外在因素和视野。

用户可以设置每个传感器的内在参数和外在参数（位置和方向），以相对于车辆的坐标为准。请注意，CARLA使用虚幻引擎的坐标系统，即：X-前，Y-右，Z-上。
 
允许使用的传感器有：

- [**sensor.camera.rgb**](https://carla.readthedocs.io/en/0.9.10/ref_sensors/#rgb-camera) - 捕捉图像的普通相机。
- [**sensor.lidar.ray_cast**](https://carla.readthedocs.io/en/0.9.10/ref_sensors/#lidar-sensor) - Velodyne 64 激光雷达。
- [**sensor.other.radar**](https://carla.readthedocs.io/en/0.9.10/ref_sensors/#radar-sensor) - 远程雷达（最远100米）。
- [**sensor.other.gnss**](https://carla.readthedocs.io/en/0.9.10/ref_sensors/#gnss-sensor) - 返回地理位置数据的GPS传感器。
- [**sensor.other.imu**](https://carla.readthedocs.io/en/0.9.10/ref_sensors/#imu-sensor) - 六轴惯性测量单元。
- `sensor.opendrive_map` - 伪传感器，以OpenDRIVE格式解析为字符串的高清地图。
- `sensor.speedometer` - 伪传感器，提供线性速度的近似值。

> 如果您尝试使用其他传感器或传感器参数名称错误，会使传感器设置失败。
 
您可以使用这些传感器中的任何一个来配置您的传感器栈。但是，为了保持合适的计算资源负载，我们对可以添加到一个agent中的传感器数量进行了以下限制：
```bash
sensor.camera.rgb: 4
sensor.lidar.ray_cast: 1
sensor.other.radar: 2
sensor.other.gnss: 1
sensor.other.imu: 1
sensor.opendrive_map: 1
sensor.speedometer: 1
```
> 设置过多的传感器将会失败。
 
此外，还有一些空间限制，限制了传感器在车辆体积内的位置。如果一个传感器在任何轴线上与它的父体相距超过3米（例如：`[3.1,0.0,0.0]`），设置将失败。
 
 
### 2.3.4 运行时 agent 与 leaderboard 的通信
#### 2.3.4.1 覆盖 run_step 方法（基于非 ROS 的 agents）
这个方法将在每个时间步长被调用一次，产生一个新的动作，其形式为 [`carla.VehicleControl`](https://carla.readthedocs.io/en/latest/python_api/#carlavehiclecontrol) 对象。确保该函数返回控制对象，该对象将被用于更新您的 agent。
```python
def run_step(self, input_data, timestamp):
    control = self._do_something_smart(input_data, timestamp)
    return control
```
 
-  `input_data`: 一个包含所请求的传感器数据的字典。这些数据已经在`sensor_interface.py`进行了预处理，并将以 numpy 数组的形式给出。 这个字典由传感器方法中定义的id来索引。
 
- `Timestamp`：当前模拟的时间戳。
 
您也可以获取到 ego agent 为到达目的地所应走的路线。使用 `self._global_plan` 成员来访问地理定位路线，使用`self._global_plan_world_coord`来访问其在世界位置的对应。
#### 2.3.4.2 ROS 和运行时 leaderboard 的通信
ROS 和 Leaderboard之间通过同步方式集成，默认情况下，Leaderboard 会等待接收带有当前帧时间戳的车辆命令。具有重复或过时的时间戳的车辆命令将被忽略。 用户可以通过覆盖`ROSBaseAgent._vehicle_control_cmd_callback`改变. 当覆盖默认行为时，每个步骤应用哪个车辆命令应由用户负责。

车辆控制必须发布到名为`/carla/hero/vehicle_control_cmd`(carla_msgs/CarlaEgoVehicleControl)[https://github.com/carla-simulator/ros-carla-msgs/blob/leaderboard-2.0/msg/CarlaEgoVehicleControl.msg] 的 ROS topic。确保将时间戳填充消息的Header部分。

传感器数据也可通过 ROS topic获得。topic名称的结构如下：`/carla/hero/<sensor-id>`. 如需完整参考，请查看[CARLA ROS 传感器文档](https://carla.readthedocs.io/projects/ros-bridge/en/latest/ros_sensors/)。

基于 ROS 的 agent 也可以访问 ego 车辆路线。该路线发布在以下 topics 中：

- `/carla/hero/global_plan( carla_msgs/CarlaRoute )`
- `/carla/hero/global_plan_gnss( carla_msgs/CarlaGnnsRoute )`
此外，基于 ROS 的 agents 必须在堆栈初始化完成后与 Leadeboard 通信。此通信应通过在以下 topic`/carla/hero/status` [`std_msgs/Bool`](http://docs.ros.org/en/noetic/api/std_msgs/html/msg/Bool.html)中发布来完成。

### 2.3.5 覆盖destroy方法
在每条路线结束时，destroy 方法将被调用，在您需要清理的情况下，您的 agent 可以重写这个方法。例如，您可以利用这个函数来清除网络中任何不需要的储存。
```python
def destroy(self):
    pass
```

## 2.4. 训练和测试您的 agent
我们准备了一套预定义的路线，您可以使用这些路线来训练和验证您的算法的性能。路线可以在`${LEADERBOARD_ROOT}/data`文件夹中找到。
 
- `routes_training.xml`。 有15条用作训练数据的路线。
- `routes_testing.xml`。 有10条用作验证数据的路线。

---

[上一页：比赛介绍](README.md)

[下一页：比赛规则](rules.md)