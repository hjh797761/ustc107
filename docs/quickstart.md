# 快速开始

本页面向第一次使用本科生算力平台的同学。读完并操作完本页，你应当能完成一条最小闭环：进入平台、确认自己所在的位置、整理文件和环境、提交一个测试任务，并知道后续该阅读哪些页面。

本科生算力平台的入口为 [https://107.ustc.edu.cn/dashboard](https://107.ustc.edu.cn/dashboard)。目前主要通过统一身份认证和网页端访问平台。

!!! tip "本页的目标"
    本页不要求你先掌握 Linux、Slurm 或集群计算。这里先带你跑通一次最小任务；更完整的背景知识会放在基础文档和场景教程中。

## 1. 先理解平台怎么用

这个平台不是“远程桌面”，也不是每位同学独占一台云主机。更准确地说，它是一个通过网页访问的集群平台。

| 部分 | 作用 |
|---|---|
| Web Shell | 浏览器里的 Linux 终端，用来输入命令 |
| 文件管理器 | 上传、下载、移动、编辑文件 |
| 登录节点 | 登录后默认进入的入口机器，用来整理文件、配置环境、提交任务 |
| 计算节点 | 真正执行训练、推理、渲染、数据处理等任务的服务器 |
| Slurm | 作业调度系统，用来排队、分配资源和运行任务 |

!!! warning "不要在登录节点直接跑长时间任务"
    登录节点主要用于准备工作。训练、推理、渲染、批量数据处理等重计算任务，应通过 Slurm 提交到计算节点运行。这样既能保护登录节点，也方便保留日志和排查问题。

## 2. 第一次进入平台后先检查

进入平台的 Web Shell 后，先运行下面几条命令，确认当前用户、节点和目录：

```bash
whoami
hostname
pwd
echo $HOME
```

你应该能看到自己的用户名、当前节点名、当前目录和 home 目录。

接着检查 Slurm 命令是否可用：

```bash
which sinfo
which sbatch
which squeue
```

如果这些命令能找到，说明你可以使用 Slurm 查看资源、提交任务和查看队列。

## 3. 建议先整理好目录

为了后续不把文件堆乱，建议先在 home 目录下建立几个常用目录：

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
| `~/projects` | 课程作业、训练代码和实验项目 |
| `~/envs` | 自己创建的 Python 环境 |
| `~/logs` | Slurm 任务输出和错误日志 |
| `~/jobs` | Slurm 作业脚本 |
| `~/checkpoints` | 模型权重和中间结果 |

创建完成后检查：

```bash
ls ~
```

如果能看到这些目录，说明准备完成。

## 4. 上传项目文件

第一次使用时，推荐通过平台的文件管理器上传文件。

如果项目是一个文件夹，建议先在本地电脑上打包成 `.zip`、`.tar` 或 `.tar.gz`，再上传到 `~/projects`。这样通常比逐个上传文件更稳定，也更容易确认有没有漏文件。

上传后，在 Web Shell 中检查：

```bash
cd ~/projects
ls
```

如果能看到上传的压缩包或项目文件夹，说明上传成功。

常见解压命令如下：

```bash
# 解压 .tar.gz 文件
tar -xzvf <文件名>.tar.gz

# 解包 .tar 文件
tar -xvf <文件名>.tar

# 解压 .zip 文件
unzip <文件名>.zip
```

## 5. 准备自己的 Python 环境

目前平台不提供统一可用的 conda 环境。第一次使用时，要先准备一个属于自己的 Python 环境。

如果你的项目依赖较简单，可以先使用 Python 自带的 `venv`：

```bash
python3 -m venv ~/envs/myenv
source ~/envs/myenv/bin/activate
python -m pip install --upgrade pip
```

激活成功后，命令行前面通常会出现环境名，例如：

```text
(myenv) ...
```

也可以用下面的命令确认当前 Python 来自你的环境：

```bash
which python
python -V
```

之后进入项目目录安装依赖：

```bash
cd ~/projects/<你的项目目录>
pip install -r requirements.txt
```

如果你的项目没有 `requirements.txt`，请按照课程或项目说明安装对应依赖。若项目依赖较复杂，也可以使用 Miniforge、mamba 或 conda；相关内容见 [环境配置](basics/environments.md)。

## 6. 提交第一个测试任务

平台统一认证登录的同学，默认使用 `Students` 分区和 `qos_stu_default`，初始算力配额为 `4CPU / 1GPU / 4h`。如果课程说明、平台页面或管理员通知有新的要求，请以最新说明为准。

先创建日志和脚本目录：

```bash
mkdir -p ~/jobs ~/logs
```

新建文件 `~/jobs/hello.slurm`，写入：

```bash
#!/bin/bash
#SBATCH --job-name=hello
#SBATCH --partition=Students
#SBATCH --qos=qos_stu_default
#SBATCH --gres=gpu:1
#SBATCH --cpus-per-task=4
#SBATCH --output=/home/scc/%u/logs/hello_%j.out
#SBATCH --error=/home/scc/%u/logs/hello_%j.err

echo "Hello from Slurm"
hostname
date
```

这里的几个参数含义是：

| 参数 | 含义 |
|---|---|
| `--partition=Students` | 使用学生分区 |
| `--qos=qos_stu_default` | 使用统一认证学生默认 QoS |
| `--gres=gpu:1` | 申请 1 块 GPU |
| `--cpus-per-task=4` | 申请 4 个 CPU 核心 |
| `%u` | 当前用户名 |
| `%j` | Slurm 作业号 |

提交任务：

```bash
sbatch ~/jobs/hello.slurm
```

如果提交成功，终端会返回一个作业号，例如：

```text
Submitted batch job 12345
```

查看自己的任务：

```bash
squeue -u $USER
```

任务结束后，到 `~/logs` 查看输出。把下面命令中的 `<作业号>` 换成实际作业号：

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

如果你需要更长运行时间、更多 CPU、更多 GPU 或多卡任务，可以通过平台申请额外算力：[https://107.ustc.edu.cn/apply](https://107.ustc.edu.cn/apply)。
