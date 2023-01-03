[上一页：比赛报名](https://seekerxc.github.io/#/register)

---

## 1. 环境准备
在${LEADERBOARD_ROOT}文件夹外创建一个新文件夹，命名自定义（例如team_code），定义环境变量：
export TEAM_CODE_ROOT=.../team_code
将你的autonomous agent以及相关的代码放入${TEAM_CODE_ROOT}文件夹中。

> 注意：你对于CARLA、Leaderboard或Scenario Runner的任何修改在提交后将不会被应用和运行。

打开用于构建镜像的文件：
```shell
vim ${LEADERBOARD_ROOT}/scripts/Dockerfile.master
```
对于基于ROS的agent：
```shell
vim ${LEADERBOARD_ROOT}/scripts/Dockerfile.ros
```
在dockerfile中，修改如下内容，以指定你的agent路径：


## 2. 生成镜像
在构建镜像前，确保定义了以下的环境变量：
```shell
export CARLA_ROOT=...
export SCENARIO_RUNNER_ROOT=...
export LEADERBOARD_ROOT=...
export TEAM_CODE_ROOT=...
export CARLA_ROS_BRIDGE_ROOT=... # Optional: Only for ROS based agents
```
然后执行以下的脚本构建镜像：
```shell
${LEADERBOARD_ROOT}/scripts/make_docker.sh
```
构建的镜像命名为leaderboard-user，可以在docker镜像中找到，可以通过在电脑本地运行leaderboard来测试docker镜像。

## 3. 提交镜像
- 进入[比赛报名系统](http://161.189.217.21:3000/)
- 点击"Contests"，选择“carsmos 2023 春季赛事”
- 点击“Submissions”，创建一个新的Submission
- 点击"Get instructions“获取提交镜像的指令
- 按照获取的指令提交镜像

