
# SMART-LLM 操作笔记

## 1. 项目说明

`SMART-LLM` 是一个基于大模型的多机器人任务规划框架，主要流程是：

- `run_llm.py` 负责生成任务分解、任务分配和代码
- `execute_plan.py` 负责把生成结果编译成可执行脚本并运行 AI2Thor
- `data/aithor_connect/` 负责 AI2Thor 的实际动作执行、可视化和视频保存

项目执行时依赖 AI2Thor，因此在 Windows 上不能直接原生运行，需要通过 **Ubuntu WSL + Docker Desktop** 配合完成。

---

## 2. 环境搭建

### 2.1 安装 Docker Desktop

目的：
- 提供容器运行环境
- 支持 GPU 透传
- 为 AI2Thor 的 Docker 执行做准备

操作：
- 安装 Docker Desktop
- 在设置中启用 `WSL2 backend`
- 确认 NVIDIA 驱动已更新到支持 WSL2 GPU 的版本

验证命令：
```powershell
docker run --rm -it --gpus=all nvcr.io/nvidia/k8s/cuda-sample:nbody nbody -gpu -benchmark
```

如果成功：
- 能看到你的 NVIDIA GPU
- benchmark 正常输出
- 说明 Docker + WSL2 + GPU 通路正常

---

### 2.2 安装 Ubuntu WSL

目的：
- 给 `.sh` 脚本和 Linux 环境一个真正可执行的 shell
- 让 AI2Thor 的运行脚本可以在 Linux 环境里执行

操作：
- 打开 PowerShell（管理员）
- 执行：

```powershell
wsl --install -d Ubuntu
```

- 按提示重启
- 从开始菜单打开 Ubuntu
- 第一次启动时创建 Linux 用户名和密码

---

### 2.3 安装基础工具

目的：
- 安装 `git`、`python3`、`pip`、`venv`
- 为后续安装依赖和运行脚本做准备

命令：
```bash
sudo apt update
sudo apt install -y git python3 python3-pip python3-venv
```

---

### 2.4 安装图形和视频依赖

目的：
- 解决 OpenCV / Qt 弹窗问题
- 解决视频生成所需的 `ffmpeg`

命令：
```bash
sudo apt update
sudo apt install -y \
  libsm6 \
  libice6 \
  libxcb-cursor0 \
  libxcb-xinerama0 \
  libxcb-icccm4 \
  libxcb-image0 \
  libxcb-keysyms1 \
  libxcb-render-util0 \
  libxkbcommon-x11-0 \
  ffmpeg
```

说明：
- `libsm6` 和 `libice6` 是 Qt/xcb 常见缺失依赖
- `ffmpeg` 用于把保存的帧图片生成视频

---

## 3. 项目目录

推荐把频繁运行的项目放在 WSL 的 Linux 文件系统里，例如：

```bash
/home/proudleafye/projects/
```

如果你的实际根目录是 `~/projects`，也可以直接用它。

### 3.1 创建工作目录

命令：
```bash
mkdir -p ~/projects
```

### 3.2 复制项目到 WSL

如果项目在 Windows 的 `E:\SMART-LLM`，可以复制到 WSL：

```bash
cp -r /mnt/e/SMART-LLM ~/projects/
```

如果 `ai2thor-docker` 也要一起用：

```bash
cp -r /mnt/e/ai2thor-docker ~/projects/
```

### 3.3 进入项目目录

命令：
```bash
cd ~/projects/SMART-LLM
```

如果你的项目根目录就是 `~/projects`，则：

```bash
cd ~/projects
```

---

## 4. Python 环境

### 4.1 创建虚拟环境

目的：
- 隔离 Python 依赖
- 避免污染系统环境

命令：
```bash
cd ~/projects/SMART-LLM
python3 -m venv .venv
source .venv/bin/activate
```

### 4.2 安装依赖

命令：
```bash
pip install -r requirments.txt
```

说明：
- 项目里文件名是 `requirments.txt`
- 不是常见的 `requirements.txt`

---

## 5. LLM 配置

项目已支持多模型调用：
- OpenAI
- DeepSeek
- Gemini

### 5.1 OpenAI 示例

命令：
```bash
python3 scripts/run_llm.py --floor-plan 6 --provider openai --gpt-version gpt-4o-mini --api-key-file api_key
```

### 5.2 DeepSeek 示例

命令：
```bash
python3 scripts/run_llm.py --floor-plan 6 --provider deepseek --gpt-version deepseek-chat --api-key-file deepseek_api_key
```

### 5.3 Gemini 示例

命令：
```bash
python3 scripts/run_llm.py --floor-plan 6 --provider gemini --gpt-version gemini-2.0-flash --api-key-file gemini_api_key
```

说明：
- `--provider` 指定模型提供方
- `--gpt-version` 实际上是模型名
- `--api-key-file` 指定密钥文件

---

## 6. 运行流程

### 6.1 生成计划

目的：
- 用 LLM 对任务进行分解、分配、代码生成
- 结果会保存到 `logs/` 下

命令：
```bash
python3 scripts/run_llm.py --floor-plan 6
```

执行后会看到类似：
- `Generating Decompsed Plans...`
- `Generating Allocation Solution...`
- `Generating Allocated Code...`

然后在 `logs/` 下生成任务目录，例如：

```text
Slice_the_tomato_plans_03-20-2026-10-49-42
```

---

### 6.2 执行计划

目的：
- 把 `logs/` 下生成的代码编译成 `executable_plan.py`
- 真正启动 AI2Thor 执行任务

命令：
```bash
python3 scripts/execute_plan.py --command Slice_the_tomato_plans_03-20-2026-10-49-42/
```

说明：
- `--command` 后面填写 `logs/` 里的任务目录
- 目录名末尾可以带 `/`
- 如果是在 Windows 原生 shell 下执行，会报错，必须在 Ubuntu WSL 中运行

---

## 7. 可视化与输出

执行时会出现：

- 每个 agent 的第一视角窗口
- 一个 Top View 总览窗口
- 图片保存到任务目录下
- 可能生成视频文件

相关文件位置：
- `data/aithor_connect/aithor_connect.py`
- `data/aithor_connect/imports_aux_fn.py`

### 7.1 agent 窗口
- 每个 agent 会弹出一个 OpenCV 窗口
- `cv2.imshow('agent%s' % i, e.cv2img)` 会显示第一视角

### 7.2 顶视图窗口
- `cv2.imshow('Top View', top_view_rgb)` 会显示顶视图

### 7.3 保存图片
- 每一步都会保存图片到：
  - `agent_1/`
  - `agent_2/`
  - `top_view/`

### 7.4 生成视频
- 通过 `ffmpeg` 把图片序列拼成视频
- 如果没装 `ffmpeg`，现在会跳过视频生成，不会中断整个流程

---

## 8. 你遇到过的典型问题与修复

### 8.1 Windows 里运行 `.sh` 脚本失败

现象：
- `'.' is not recognized as an internal or external command`

原因：
- `.sh` 是 Linux shell 脚本
- 不能直接在 Windows CMD/PowerShell 里运行

解决：
- 用 Ubuntu WSL 运行
- 或者用 Git Bash，但 AI2Thor 的宿主检查更适合在 Ubuntu WSL 里跑

---

### 8.2 `build.sh` 报 Nvidia driver not found

现象：
- `Error: Nvidia driver not found at /proc/driver/nvidia/version`

原因：
- `ai2thor-docker/scripts/build.sh` 是按传统 Linux 宿主写的
- Git Bash / Windows shell 不具备这个路径
- WSL 下也不完全匹配它的宿主检查逻辑

解决：
- 不把 `build.sh` 当作当前主路径
- 优先用 Ubuntu WSL 直接运行项目
- 必要时手动构建 Docker 镜像

---

### 8.3 Qt / xcb 报错

现象：
- `Could not load the Qt platform plugin "xcb"`
- `QFontDatabase: Cannot find font directory ...`

原因：
- 缺少 Linux 图形依赖
- OpenCV 的 Qt 插件需要额外系统库

解决：
```bash
sudo apt update
sudo apt install -y \
  libsm6 \
  libice6 \
  libxcb-cursor0 \
  libxcb-xinerama0 \
  libxcb-icccm4 \
  libxcb-image0 \
  libxcb-keysyms1 \
  libxcb-render-util0 \
  libxkbcommon-x11-0
```

说明：
- 字体警告可以先忽略
- 只要窗口能正常显示，通常不影响执行

---

### 8.4 找不到 ffmpeg

现象：
- `FileNotFoundError: [Errno 2] No such file or directory: 'ffmpeg'`

原因：
- 系统没有安装 `ffmpeg`
- 或者当前 shell 没刷新 PATH

解决：
```bash
sudo apt update
sudo apt install -y ffmpeg
```

现在项目里已经改成：
- 找不到 `ffmpeg` 就跳过视频生成
- 不再让任务最后一步报错

---

### 8.5 API key 401

现象：
- `HTTP Error 401: Unauthorized`
- `Incorrect API key provided`

原因：
- `api_key.txt` 中的 key 无效、过期或 provider 不匹配

解决：
- 检查 key 是否有效
- 根据 provider 选择对应 key
- 确认没有多余空格或引号

---

## 9. 当前执行逻辑理解

### 9.1 规划阶段
在 `run_llm.py` 里，LLM 负责：

- 任务分解
- 任务分配
- 代码生成

### 9.2 执行阶段
在 `execute_plan.py` 里：

- 读取 `logs/` 下的计划
- 拼接成 `executable_plan.py`
- 调用 AI2Thor 执行

### 9.3 运行时观测
执行阶段会实时读取：

- AI2Thor 的场景 metadata
- 物体位置
- 机器人位置
- 视觉帧

但这些信息主要是给本地控制逻辑用的，**不是回传给 LLM 的闭环观测**。

---

## 10. 常用命令速查

### 10.1 进入项目
```bash
cd ~/projects/SMART-LLM
```

### 10.2 激活虚拟环境
```bash
source .venv/bin/activate
```

### 10.3 生成计划
```bash
python3 scripts/run_llm.py --floor-plan 6
```

### 10.4 执行任务
```bash
python3 scripts/execute_plan.py --command Slice_the_tomato_plans_03-20-2026-10-49-42/
```

### 10.5 检查 GPU
```bash
nvidia-smi
```

### 10.6 检查 ffmpeg
```bash
command -v ffmpeg
ffmpeg -version | head -n 2
```

---

## 11. 关键文件位置

- `scripts/run_llm.py`
- `scripts/execute_plan.py`
- `scripts/llm_client.py`
- `data/aithor_connect/aithor_connect.py`
- `data/aithor_connect/imports_aux_fn.py`
- `logs/<task_name>/`

---

## 12. 总结

你的实际工作流可以记成一句话：

- **Windows 负责开发**
- **Ubuntu WSL 负责运行**
- **Docker Desktop 负责容器和 GPU 支持**
- **`run_llm.py` 生成计划**
- **`execute_plan.py` 执行任务**
- **`aithor_connect.py` 负责 AI2Thor 动作和可视化**

---

如果你愿意，我下一步可以继续把这份内容整理成更像项目文档的版本，改成：

1. `环境搭建`
2. `运行指南`
3. `常见问题`
4. `开发说明`

这种更正式的 README 样式。