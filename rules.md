[上一页：开发环境](install.md)

---

## 3.1. 比赛路线
自动驾驶车辆根据路线描述信息，达到目的地。路线描述信息是一个元组列表。 元组的第一个元素包含一个航点，用纬度、经度和Z分量表示。 这里是使用世界坐标来表示路线。
```bash
[({'x': 153.7, 'y': 15.6, 'z': 0.0}, RoadOption.LEFT),
 ({'x': 148.9, 'y': 67.8, 'z': 0.0}, RoadOption.RIGHT),
 ...
 ({'x': 180.7, 'y': 45.1, 'z': 1.2}, RoadOption.STRAIGHT)]
 ```
?> 友情提示: 两个连续的航点之间的距离可能有数百米。请避免使用这些信息作为您的算法参数。

第二个元素包含一个高级命令。可用的高级命令集如下：

```python
RoadOption.CHANGELANELEFT：向左移动一条车道。
RoadOption.CHANGELANERIGHT: 向右移动一条车道。
RoadOption.LANEFOLLOW: 继续在当前车道行驶。
RoadOption.LEFT: 在十字路口左转。
RoadOption.RIGHT: 在路口右转。
RoadOption.STRAIGHT: 在十字路口保持直行。
```

可能有这样的情况：左和右的语义是模糊的。为了消除这些情况，你可以考虑下一个航点的GPS位置。

!> <span style="color: red; font-weight: 1000; font-size: 150%; text-underline-position: below; text-decoration: underline;">重要提示：不允许利用Openatom Carsmos全球开源自动驾驶算法大赛提供的任何非授权信息，包括任何类型的真值。 使用了这些信息的提交将被作废，团队将被禁止参加比赛。</span>


自动驾驶车辆可以请求访问以下传感器。此外，自动驾驶车辆可以请求访问高清地图，该地图以OpenDRIVE文件的形式提供，并被解析为一个字符串。
你可以将该文件解析或转换为对你的agent有用的表示。

```python
sensor.other.gnss
sensor.other.imu
sensor.lidar.ray_cast
sensor.other.radar
sensor.camera.rgb
sensor.other.speedometer
```

```bash
GPS传感器返回地理位置数据
6轴惯性测量单元
Velodyne 64线激光雷达
远程雷达（高达100米）
捕捉图像的普通相机
伪传感器，提供线速度的近似值
```

- OpenDRIVE map

- sensor.opendrive_map

- 0-1 unit

- 伪传感器，以OpenDRIVE格式解析为字符串，公开HD地图


?> 每个传感器单元可访问的数量都是有限的，这样可以保持算力需求的可控。 比赛每个提交的文件都将在AWS中使用[g4dn.4xlarge](https://aws.amazon.com/cn/ec2/instance-types/g4/?nc1=h_ls)实例进行评估。
参赛队伍在给定的（120小时时长）时间内获得有限的提交机会（目前是5次）。

!> 严禁滥用或攻击Openatom Carsmos全球开源自动驾驶算法大赛的基础设施，包括用于运行该服务的所有软件和硬件。偏离Openatom Carsmos全球开源自动驾驶算法大赛精神的行为可能导致团队被禁止参赛。

## 3.2. 评分规则
自动驾驶车辆的驾驶能力可以用多个指标来描述。在这次的Openatom Carsmos全球开源自动驾驶算法大赛中，我们选择了一套有助于了解驾驶的不同方面的指标。虽然所有路线都有相同类型的指标，但它们各自的数值是单独计算的。具体指标如下:
- **行驶得分**：比赛的主要指标，是路线完成度和违章处罚的乘积。这里是第条路线的完成率，是第条路线的违章处罚。
- **路线完成度**：Agent完成的路线距离占完整路线距离的百分比。
- **违规处罚**：本次比赛追踪几种类型的违规行为，这个指标将agent触发的所有违规行为汇总为一个几何序列。初始状态为一个理想的1.0基本分，每一种违规行为都会扣除一定的分数。
当所有的路线都完成后，会产生前文提及三种类型的全局指标，它是所有单独路线的算术平均值。
>全局行驶得分是主要评价指标，比赛将根据全局行驶得分将你与其他参赛者进行区分。

## 3.3. 违章和停止运行事件
本次大赛提供了一系列违规行为的独立评判指标。每种违规行为都有一个惩罚系数，每次发生时都会适用。按严重程度排序，这些违规行为如下：
- **与人的碰撞 — 0.50**
- **与其他车辆的碰撞 — 0.60**
- **与静态元素的碰撞 — 0.65**
- **闯红灯 — 0.70**
- **闯过stop标志 — 0.80**

一些场景的特点是可以无限期地阻止被测车辆的行为。这些场景将有一个4分钟的超时，超时后被测车辆将被释放并继续行驶。然而，当违反时间限制时，会有惩罚：
- **场景超时 — 0.7**
  
Agent被期望保持一个与附近交通一致的最低车速。Agent的速度将与附近车辆的速度进行比较，如果不能保持一个合适的车速，将导致处罚。处罚将取决于速度差异的大小，最高为以下数值：
- **未保持最低速度 — 0.7**

Agent应该避让从后方靠近的紧急车辆。如果紧急车辆无法通过，将被处罚：
- **未避让紧急车辆 — 0.7**

除此以外，还有一种没有系数的违规行为，会影响路线完成度的计算。(Ri)
- **非公路驾驶** - 如果agent在非公路上驾驶，该百分比的路线将不被用于计算路线完成度得分。

此外，有些事件会中断模拟进程，阻止agent继续行驶。在这些情况下，正在模拟的路线将被关闭，直接转到下一条路线，正常触发：
- **路线偏离** — 如果agent偏离指定路线超过30米。
- **程序超时** — 如果在180秒模拟时间内agent没有采取任何行动。
- **模拟超时** — 如果在60秒内无法建立客户端-服务器通信。
- **路线超时** — 如果路线需要很长时间才能完成。

每次发生上述任何情况，都会记录一些细节，这些细节将显示为一个列表，供你在路线的单独指标上查看。下面是一个既闯红灯又偏离路线的例子：

```python
"infractions": {
  "Collisions with layout": [],
  "Collisions with pedestrians": [],
  "Collisions with vehicles": [],
  "Red lights infractions": [
        "Agent ran a red light 203 at (x=341.25, y=209.1, z=0.104)"
  ],
  "Stop sign infractions": [],
  "Off-road infractions": [],
  "Min speed infractions": [],
  "Yield to emergency vehicle infractions": [],
  "Scenario timeouts": [],
  "Route deviations": [
        "Agent deviated from the route at (x=95.92, y=165.673, z=0.138)"
  ],
  "Agent blocked": [],
  "Route timeouts": []
}
```
全局违章将单个路线的数据压缩成一个数值，并以每公里事件数的形式给出。

## 3.4. 场景介绍
自动驾驶车辆将面临多种交通场景，如:
- [失控](scenarios?id=_1失控)
- [交通避让](scenarios?id=_2交通避让)
- [高速公路](scenarios?id=_3高速公路)
- [避障](scenarios?id=_4避开障碍物)
- [刹车和变道](scenarios?id=_5刹车和变道)
- [停车](scenarios?id=_5停车)

更完整的交通场景说明可以在 [**比赛交通场景**](scenarios.md) 中查看。

---

[上一页：开发环境](install.md)

[下一页：提交说明](submit.md)