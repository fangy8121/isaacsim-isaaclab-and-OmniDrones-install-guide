# Isaac Sim / Isaac Lab / OmniDrones / OmniPerception 安装记录

本文记录 Isaac Sim、Isaac Lab、OmniDrones 以及 LiDAR 模块的安装过程。Isaac Sim 和 Isaac Lab 的安装主要参考 Isaac Lab v2.3.2 官方 pip 安装文档；OmniDrones 使用兼容 Isaac Sim 5.1.0 的 [ChanJoon/OmniDrones](https://github.com/ChanJoon/OmniDrones)；LiDAR 后续使用 [aCodeDog/OmniPerception](https://github.com/aCodeDog/OmniPerception)。

## 参考资料

- Isaac Lab v2.3.2 pip 安装文档：<https://isaac-sim.github.io/IsaacLab/v2.3.2/source/setup/installation/pip_installation.html>
- OmniDrones 官方安装文档：<https://omnidrones.readthedocs.io/en/latest/installation.html>
- ChanJoon/OmniDrones：<https://github.com/ChanJoon/OmniDrones>
- aCodeDog/OmniPerception：<https://github.com/aCodeDog/OmniPerception>

## 版本组合

| 组件 | 版本 |
| --- | --- |
| Python | 3.11 |
| Isaac Sim | 5.1.0 |
| Isaac Lab | v2.3.2 |
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

克隆 Isaac Lab，并进入仓库查看 helper script\
此处需要把版本固定为 v2.3.2，避免现在 GitHub 的 main 将来变成其他版本：
```bash
git clone --branch v2.3.2 https://github.com/isaac-sim/IsaacLab.git
cd IsaacLab
./isaaclab.sh --help
```

安装 Linux 依赖并安装 Isaac Lab：

```bash
sudo apt install cmake build-essential
./isaaclab.sh --install
```

（建议）如果只想安装某个学习框架，例如 `rsl_rl`：

```bash
./isaaclab.sh --install rsl_rl
```

验证 Isaac Lab：

```bash
./isaaclab.sh -p scripts/tutorials/00_sim/create_empty.py
# 或者直接使用当前 Python 环境运行
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

直接执行 `pip install -e .` 时，pip 可能会根据 `setup.py` 中的依赖范围自动升级部分包，从而出现版本不兼容问题。下面先按已验证成功的环境固定关键依赖版本：

```bash
pip install \
    tensordict==0.10.0 \
    torchrl==0.10.0

pip install --force-reinstall --no-deps packaging==23.0
```

然后安装 OmniDrones。本步骤使用 `--no-deps`，避免 pip 再次自动修改上面已经固定好的依赖版本：

```bash
pip install -e . --no-deps
```

（建议）验证安装，运行一个较小的 Hover + PPO 测试：

```bash
cd ~/OmniDrones/scripts
python train.py \
    task=Hover \
    algo=ppo \
    headless=true \
    task.env.num_envs=8 \
    wandb.mode=disabled \
    total_frames=10000
```
或者直接按照原文测试：

```bash
python train.py algo=ppo headless=true wandb.mode=disabled
```
如果 PPO 可以正常训练，但训练结束出现：

```bash
IndexError: too many indices for array:
array is 1-dimensional, but 3 were indexed
```
则修改~/OmniDrones/omni_drones/envs/isaac_env.py：

```bash
rgb_data = self._rgb_annotator.get_data()
rgb_data = np.frombuffer(rgb_data, dtype=np.uint8).reshape(*rgb_data.shape)
return rgb_data[:, :, :3]
```
改为：

```bash
rgb_data = self._rgb_annotator.get_data()

if rgb_data is None:
    width, height = self.cfg.viewer.resolution
    return np.zeros((height, width, 3), dtype=np.uint8)

rgb_array = np.asarray(rgb_data)

if rgb_array.size == 0 or rgb_array.ndim < 3:
    width, height = self.cfg.viewer.resolution
    return np.zeros((height, width, 3), dtype=np.uint8)

rgb_data = np.frombuffer(
    rgb_data, dtype=np.uint8
).reshape(*rgb_data.shape)

return rgb_data[:, :, :3]
```

## 5. 安装 LiDAR：OmniPerception

LiDAR 后续使用 [aCodeDog/OmniPerception](https://github.com/aCodeDog/OmniPerception)。该项目提供 `LidarSensor` 模块，支持 Livox、Velodyne、Ouster 等多种 LiDAR pattern，并包含 IsaacLab / Isaac Sim 等平台的集成说明。
可以使用原文安装，但是我更推荐使用本文的方式：
1. 下载 OmniPerception：

```bash
cd ~
git clone https://github.com/aCodeDog/OmniPerception.git
cd ~/OmniPerception
```
2. 安装 LidarSensor：

```bash
conda activate env_isaaclab
cd ~/OmniPerception/LidarSensor
pip install -e . --no-deps
```
3. 集成到 Isaac Lab：

```bash
cd ~/OmniPerception/LidarSensor/LidarSensor/example/isaaclab/isaaclab
./install_lidar_sensor.sh ~/IsaacLab
```
4. 测试 OmniPerception：

```bash
cd ~/IsaacLab
./isaaclab.sh -p scripts/demos/simple_lidar_integration.py
```
如果 Isaac Sim 可以正常启动，并且场景中可以看到 LiDAR 扫描点云，说明 OmniPerception 已经成功安装。

注意：OmniPerception README 中写到 Isaac Sim 支持范围为 `<= 4.5`，并提示 Isaac Sim 5.0 暂不支持。同时，Issue #29 中有人反馈不同 Isaac Sim 版本下 LiDAR 表现可能不一致，例如 Isaac Sim 4.5 表现正常，而 Isaac Sim 5.0 中运动物体场景下 LiDAR 可能失效。因此如果在 Isaac Sim 5.1.0 环境中使用 OmniPerception，需要额外验证 LiDAR 在目标任务中的实际效果。

## 6. 其他问题

如果安装过程中遇到其他问题，请优先查看上方列出的源文档和对应 GitHub Issue。
