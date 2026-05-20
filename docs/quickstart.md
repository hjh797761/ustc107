# 快速开始

本页面向第一次使用本科生算力平台的同学，目标是帮你先跑通一条最小流程：进入平台、确认环境、整理文件、提交一个测试任务，并知道后续该阅读哪些页面。

如果你只是想尽快完成课程作业，建议先按本页走一遍；如果你想理解每一步背后的原因，再继续阅读基础文档和场景教程。

## 1. 先理解平台怎么用

本科生算力平台不是“远程桌面”，也不是每个人独占一台云主机。更准确地说，它由几部分一起工作：

| 部分 | 作用 |
|---|---|
| Web Shell | 浏览器里的 Linux 终端 |
| 文件管理器 | 上传、下载、移动、编辑文件 |
| Slurm 作业系统 | 把训练或计算任务提交到计算节点运行 |
| 计算节点 | 真正执行训练、推理、渲染、数据处理等任务的服务器 |

!!! warning "不要在登录节点直接跑长时间任务"
    登录节点主要用于登录、整理文件、配置环境和提交任务。正式训练、推理、渲染等重计算任务应通过 Slurm 提交到计算节点运行。

## 2. 第一次进入平台后先检查

进入平台的 Web Shell 后，先运行下面几条命令，确认自己在哪里：

```bash
whoami
hostname
pwd
echo $HOME
```

你应该能看到自己的用户名、当前节点名、当前目录和 home 目录。

如果平台已经配置 Slurm，可以继续检查：

```bash
which sinfo
which sbatch
which squeue
```

如果这些命令能找到，说明可以使用 Slurm 查看资源、提交任务和查看队列。

## 3. 建议先整理好目录

为了后续不把文件堆乱，建议先在 home 目录下建立几个常用文件夹：

```bash
mkdir -p ~/projects
mkdir -p ~/envs
mkdir -p ~/logs
mkdir -p ~/jobs
mkdir -p ~/checkpoints
```

推荐用途如下：

| 目录 | 用途 |
|---|---|
| `~/projects` | 放课程作业、训练代码和实验项目 |
| `~/envs` | 放 Python 虚拟环境或环境相关文件 |
| `~/logs` | 放任务输出日志 |
| `~/jobs` | 放 Slurm 作业脚本 |
| `~/checkpoints` | 放模型权重和中间结果 |

创建完成后可以检查：

```bash
ls ~
```

如果能看到这些目录，说明准备完成。

## 4. 上传你的项目文件

第一次使用时，推荐先通过平台的文件管理器上传文件。

如果项目是一个文件夹，建议先在本地电脑上打包成 `.zip`、`.tar` 或 `.tar.gz`，再上传到 `~/projects`。这样通常比一个个文件上传更稳定，也更容易确认有没有漏文件。

上传后，可以在 Web Shell 中进入目录检查：

```bash
cd ~/projects
ls
```

如果能看到你上传的压缩包或项目文件夹，说明上传成功。

如果你上传的是 `.tar.gz` 文件，可以解压：

```bash
tar -xzvf <文件名>.tar.gz
```

如果你上传的是 `.tar` 文件，可以解包：

```bash
tar -xvf <文件名>.tar
```

如果你上传的是 `.zip` 文件，可以尝试：

```bash
unzip <文件名>.zip
```

## 5. 准备 Python 环境

如果你的任务需要 Python，可以先创建一个用户自己的虚拟环境：

```bash
python3 -m venv ~/envs/myenv
source ~/envs/myenv/bin/activate
python -m pip install --upgrade pip
```

激活成功后，命令行前面通常会出现环境名，例如：

```text
(myenv) ...
```

也可以用下面的命令确认当前 Python 来自你的虚拟环境：

```bash
which python
python -V
```

之后可以进入项目目录安装依赖：

```bash
cd ~/projects/<你的项目目录>
pip install -r requirements.txt
```

如果你的项目没有 `requirements.txt`，请按照课程或项目说明安装对应依赖。

## 6. 提交一个测试任务

在平台上运行正式任务时，推荐写 Slurm 作业脚本，然后用 `sbatch` 提交。

可以先在 `~/jobs` 下创建一个简单测试脚本，用来确认作业能提交、日志能生成：

```bash
mkdir -p ~/jobs ~/logs
```

新建文件 `~/jobs/hello.slurm`，写入：

```bash
#!/bin/bash
#SBATCH --job-name=hello
#SBATCH --output=/home/scc/%u/logs/hello_%j.out
#SBATCH --error=/home/scc/%u/logs/hello_%j.err

echo "Hello from Slurm"
hostname
date
```

提交任务：

```bash
sbatch ~/jobs/hello.slurm
```

如果提交成功，终端会返回一个作业号，例如：

```text
Submitted batch job 12345
```

可以用下面的命令查看自己的任务：

```bash
squeue -u $USER
```

任务结束后，到 `~/logs` 查看输出。把下面命令里的 `<作业号>` 换成实际作业号：

```bash
ls ~/logs
cat ~/logs/hello_<作业号>.out
```

如果日志中能看到 `Hello from Slurm`、节点名和时间，说明你已经完成了一次最小测试任务。

## 7. 后续阅读路径

如果你要完成深度学习课程作业，请继续阅读：

- [使用集群提交深度学习作业任务](guides/ai/deep-learning-homework.md)

如果你想理解基础操作，请继续阅读：

- [GUI 使用](basics/gui.md)
- [命令行使用](basics/cli.md)
- [环境配置](basics/environments.md)
- [提交任务](basics/jobs.md)
- [Slurm 速查](basics/slurm.md)
- [常见问题](basics/faq.md)