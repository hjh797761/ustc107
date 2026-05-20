# 提交任务

本页面介绍如何在平台上提交计算任务。你将了解两种常见方式：用于临时调试的交互式任务，以及用于正式运行的批处理任务。同时，本页也会说明常见资源申请参数、日志查看和失败排查方法。

## 1. 为什么需要提交任务

本科生算力平台由登录节点和计算节点共同组成。登录节点用于登录平台、整理文件、配置环境和提交任务；计算节点才是真正运行训练、推理、渲染、数据处理等任务的地方。

!!! warning "不要在登录节点直接跑长时间任务"
    如果在登录节点直接运行长时间或高负载程序，可能影响其他同学正常使用平台。正式计算任务应通过 Slurm 提交到计算节点运行。

在平台上运行任务，一般有两种方式：

| 方式 | 适合场景 | 特点 |
|---|---|---|
| 交互式任务 | 临时调试、检查环境、短时间试运行 | 像进入一台计算节点临时操作，适合探索和排错 |
| 批处理任务 | 正式训练、长时间计算、可复现实验 | 写好脚本后提交排队运行，适合保留日志和重复执行 |

## 2. 交互式任务

交互式任务适合在正式提交前做短时间检查，例如：

- 确认 Python 环境能否激活；
- 检查依赖是否安装完整；
- 测试一小段代码能否运行；
- 查看任务实际运行在哪个节点；
- 在计算节点上运行 `nvidia-smi` 检查 GPU。

交互式任务通常先申请资源，再进入一个可以执行命令的 shell。当前平台示例中，统一认证登录的同学默认使用 `Students` 分区和 `qos_stu_default`。

```bash
salloc --partition=Students --qos=qos_stu_default --gres=gpu:1 --cpus-per-task=4 --time=00:30:00
srun --pty bash
```

进入计算节点后可以运行：

```bash
hostname
whoami
pwd
nvidia-smi
```

用完交互式任务后，记得退出：

```bash
exit
```

!!! tip "交互式任务适合调试，不适合长期占用"
    如果任务已经能稳定运行，建议改成批处理脚本提交。这样更容易保留日志，也更方便复现实验。

## 3. 批处理任务

批处理任务是更推荐的正式运行方式。你把要执行的命令写进一个 `.slurm` 或 `.sbatch` 文件，然后用 `sbatch` 提交给 Slurm。

下面是一个最小示例：

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

可以把它保存为：

```bash
~/jobs/hello.slurm
```

提交前先确认脚本存在：

```bash
ls ~/jobs
```

提交任务：

```bash
sbatch ~/jobs/hello.slurm
```

如果提交成功，终端会输出类似：

```text
Submitted batch job 12345
```

其中 `12345` 是作业号。后续查看日志、取消任务时经常会用到它。

## 4. 查看和取消任务

查看自己当前的任务：

```bash
squeue -u $USER
```

常见状态包括：

| 状态 | 含义 |
|---|---|
| `PD` | Pending，正在排队，还没有开始运行 |
| `R` | Running，正在运行 |
| `CG` | Completing，正在收尾 |
| 查询不到 | 任务可能已经结束、失败或被取消 |

如果发现任务提交错了、一直排队不需要了，或者程序可能卡住了，可以取消任务：

```bash
scancel <作业号>
```

例如：

```bash
scancel 12345
```

取消后可以再次查看队列：

```bash
squeue -u $USER
```

如果列表里已经没有这个作业，说明任务已经结束或取消成功。

## 5. 常见资源申请参数

批处理脚本开头的 `#SBATCH` 行用于申请资源和设置任务属性。统一认证登录的同学，当前默认使用 `Students` 分区、`qos_stu_default`，初始算力配额为 `4CPU / 1GPU / 4h`。

| 参数 | 作用 | 示例 |
|---|---|---|
| `--job-name` | 设置任务名称 | `#SBATCH --job-name=test` |
| `--partition` | 指定分区 | `#SBATCH --partition=Students` |
| `--qos` | 指定 QoS | `#SBATCH --qos=qos_stu_default` |
| `--gres` | 申请 GPU 等通用资源 | `#SBATCH --gres=gpu:1` |
| `--cpus-per-task` | 申请 CPU 核数 | `#SBATCH --cpus-per-task=4` |
| `--time` | 申请最长运行时间 | `#SBATCH --time=04:00:00` |
| `--output` | 设置标准输出日志 | `#SBATCH --output=/home/scc/%u/logs/%x_%j.out` |
| `--error` | 设置错误日志 | `#SBATCH --error=/home/scc/%u/logs/%x_%j.err` |

如果不使用 GPU，可以在平台的“其他 sbatch 参数”中显式说明不申请 GPU，例如：

```bash
#SBATCH --gres=gpu:0
#SBATCH -c 1
```

!!! warning "资源参数不要照抄到所有任务"
    不同课程、账号、队列和时间段的资源限制可能变化。上面的参数适合作为当前学生默认用法的起点；如果平台页面、课程说明或管理员通知有新要求，请以最新说明为准。

如果需要更长运行时间、更多 CPU、更多 GPU 或多卡任务，可以通过平台申请额外算力：[https://107.ustc.edu.cn/apply](https://107.ustc.edu.cn/apply)。

## 6. 指定 GPU 类型

当前群内说明中，`anode16`、`anode17` 为 A100，其他相关 GPU 节点为 RTX 5090。需要指定 GPU 类型时，可以在资源参数中写明。

申请 1 块 RTX 5090：

```bash
#SBATCH --gres=gpu:5090:1
```

申请 1 块 A100：

```bash
#SBATCH --gres=gpu:A100:1
```

如果没有明确需求，建议先使用默认 GPU 申请方式：

```bash
#SBATCH --gres=gpu:1
```

## 7. 在作业脚本里运行 Python

如果要运行 Python 程序，通常需要在脚本中进入项目目录，并激活自己的环境。

示例：

```bash
#!/bin/bash
#SBATCH --job-name=python-test
#SBATCH --partition=Students
#SBATCH --qos=qos_stu_default
#SBATCH --gres=gpu:1
#SBATCH --cpus-per-task=4
#SBATCH --output=/home/scc/%u/logs/python_%j.out
#SBATCH --error=/home/scc/%u/logs/python_%j.err

cd ~/projects/<你的项目目录>

source ~/envs/myenv/bin/activate

which python
python -V
python main.py
```

需要根据自己的项目修改：

| 位置 | 需要改成什么 |
|---|---|
| `<你的项目目录>` | 你的实际项目文件夹 |
| `~/envs/myenv` | 你的实际 Python 环境 |
| `main.py` | 你真正要运行的 Python 文件 |

目前平台不应假设已经提供统一可用的 conda 环境；如果使用 conda、mamba 或 Miniforge，需要在作业脚本里写清楚对应的环境初始化和激活命令。

## 8. 日志与失败排查

如果脚本里写了：

```bash
#SBATCH --output=/home/scc/%u/logs/hello_%j.out
#SBATCH --error=/home/scc/%u/logs/hello_%j.err
```

任务结束后，可以查看日志目录：

```bash
ls ~/logs
```

假设作业号是 `12345`，可以查看输出日志：

```bash
cat ~/logs/hello_12345.out
```

如果程序报错，也要查看错误日志：

```bash
cat ~/logs/hello_12345.err
```

排查失败任务时，优先看错误日志最后几行：

```bash
tail -n 50 ~/logs/hello_12345.err
```

常见排查方向：

| 现象 | 可能原因 | 建议检查 |
|---|---|---|
| `QOSMaxCpuPerUserLimit` | 申请 CPU 超过当前 QoS 限制，或已有任务占用配额 | 检查 `--cpus-per-task` 和 `squeue -u $USER` |
| 找不到文件 | 当前目录不对，或路径写错 | 检查 `cd` 路径和 `ls` 输出 |
| 找不到 Python 包 | 环境没有激活，或依赖没装进当前环境 | 检查环境激活命令、`which python` 和安装记录 |
| 任务一直排队 | 资源暂时不足，或申请参数不合适 | 检查 `squeue -u $USER` 和资源参数 |
| 没有日志文件 | 日志目录不存在，或脚本没有成功提交 | 检查 `~/logs` 是否存在，确认 `sbatch` 是否返回作业号 |
| 程序中途退出 | 程序报错、资源不足或运行时间不够 | 查看 `.err` 和 `.out` 的最后部分 |

## 9. 求助前先准备这些信息

如果任务失败，需要向同学、助教或维护者求助，建议先准备：

- 你提交的作业脚本内容；
- `sbatch` 返回的作业号；
- `squeue -u $USER` 的输出；
- `.out` 日志文件；
- `.err` 错误日志文件；
- 你希望运行的命令；
- 你已经尝试过的解决方法。

这些信息能帮助别人更快判断问题出在路径、环境、资源参数，还是程序本身。
