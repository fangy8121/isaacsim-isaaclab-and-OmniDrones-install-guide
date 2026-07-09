# isaacsim/isaaclab and OmniDrones install guide

本文记录 Isaac Sim、Isaac Lab、OmniDrones 以及 LiDAR 模块的安装过程。Isaac Sim 和 Isaac Lab 的安装主要参考 Isaac Lab v2.3.0 官方 pip 安装文档；OmniDrones 使用兼容 Isaac Sim 5.1.0 的 [ChanJoon/OmniDrones](https://github.com/ChanJoon/OmniDrones)；LiDAR 后续使用 [aCodeDog/OmniPerception](https://github.com/aCodeDog/OmniPerception)。

## 参考资料

- Isaac Lab v2.3.0 pip 安装文档：<https://isaac-sim.github.io/IsaacLab/v2.3.0/source/setup/installation/pip_installation.html>
- OmniDrones 官方安装文档：<https://omnidrones.readthedocs.io/en/latest/installation.html>
- ChanJoon/OmniDrones：<https://github.com/ChanJoon/OmniDrones>
- aCodeDog/OmniPerception：<https://github.com/aCodeDog/OmniPerception>

## 版本组合

| 组件 | 版本 |
| --- | --- |
| Python | 3.11 |
| Isaac Sim | 5.1.0 |
| Isaac Lab | v2.3.0 |
| PyTorch | 2.7.0 + cu128 |
| torchvision | 0.22.0 |
| OmniDrones | ChanJoon/OmniDrones |
| LiDAR | aCodeDog/OmniPerception |

## 1. 创建 conda 环境

官方文档推荐使用 Miniconda。请先根据自己的系统安装 Miniconda，然后创建 Isaac Lab 环境：

```bash
conda create -n env_isaaclab python=3.11
conda activate env_isaaclab
```

更新 pip：

```bash
pip install --upgrade pip
```

## 2. 安装 Isaac Sim

安装 Isaac Sim pip packages：

```bash
pip install "isaacsim[all,extscache]==5.1.0" --extra-index-url https://pypi.nvidia.com
```

安装 Linux x86_64 对应的 CUDA-enabled PyTorch：

```bash
pip install -U torch==2.7.0 torchvision==0.22.0 --index-url https://download.pytorch.org/whl/cu128
```

验证 Isaac Sim：

```bash
isaacsim
```

也可以指定 experience file：

```bash
isaacsim isaacsim.exp.full.kit
```

第一次运行 Isaac Sim 时会拉取依赖扩展，并提示接受 NVIDIA Omniverse License Agreement。根据提示输入 `Yes` 即可。

## 3. 安装 Isaac Lab

克隆 Isaac Lab：

```bash
git clone https://github.com/isaac-sim/IsaacLab.git
```

进入 Isaac Lab 仓库：

```bash
cd IsaacLab
```

查看 helper script：

```bash
./isaaclab.sh --help
```

安装 Linux 依赖：

```bash
sudo apt install cmake build-essential
```

安装 Isaac Lab：

```bash
./isaaclab.sh --install
```

如果只想安装某个学习框架，例如 `rl_games`：

```bash
./isaaclab.sh --install rl_games
```

验证 Isaac Lab：

```bash
./isaaclab.sh -p scripts/tutorials/00_sim/create_empty.py
```

也可以直接使用当前 Python 环境运行：

```bash
python scripts/tutorials/00_sim/create_empty.py
```

官方文档还给出了一个快速训练示例：

```bash
./isaaclab.sh -p scripts/reinforcement_learning/rsl_rl/train.py --task=Isaac-Ant-v0 --headless
```

## 4. 安装 OmniDrones

OmniDrones 官方文档中的部分流程仍围绕旧版本 Isaac Sim / Isaac Lab 展开，并且 `conda_setup` 在 Isaac Sim 5.1.0 环境中可能不适配。因此这里使用 [ChanJoon/OmniDrones](https://github.com/ChanJoon/OmniDrones)，该 fork 已更新到 Isaac Sim 5.1.0 和 Isaac Lab v2.3.x。

下载源码：

```bash
git clone https://github.com/ChanJoon/OmniDrones.git
cd OmniDrones
```

安装：

```bash
pip install -e .
```

验证安装：

```bash
python -c "import omni_drones; print('OmniDrones installed successfully')"
```

## 5. 安装 LiDAR：OmniPerception

LiDAR 后续使用 [aCodeDog/OmniPerception](https://github.com/aCodeDog/OmniPerception)。该项目提供 `LidarSensor` 模块，支持 Livox、Velodyne、Ouster 等多种 LiDAR pattern，并包含 IsaacLab / Isaac Sim 等平台的集成说明。

安装依赖：

```bash
pip install warp-lang[extras] taichi
```

下载源码：

```bash
git clone https://github.com/aCodeDog/OmniPerception.git
cd OmniPerception
```

安装 `LidarSensor`：

```bash
cd LidarSensor
pip install -e .
```

验证安装：

```bash
python -c "import LidarSensor; print('LidarSensor installed successfully')"
```

注意：OmniPerception README 中写到 Isaac Sim 支持范围为 `<= 4.5`，并提示 Isaac Sim 5.0 暂不支持。同时，Issue #29 中有人反馈不同 Isaac Sim 版本下 LiDAR 表现可能不一致，例如 Isaac Sim 4.5 表现正常，而 Isaac Sim 5.0 中运动物体场景下 LiDAR 可能失效。因此如果在 Isaac Sim 5.1.0 环境中使用 OmniPerception，需要额外验证 LiDAR 在目标任务中的实际效果。



