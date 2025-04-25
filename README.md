# 局放图像识别系统

## 一、项目简介

本项目是一个基于机器学习的局部放电(PD)图像识别系统，使用支持向量机(SVM)算法实现对不同类型局放图像的自动分类。系统通过FastAPI框架提供RESTful API服务，可以接收上传的图像并返回识别结果。

## 二、功能特点

- 支持识别5种不同类型的局部放电图像：
  - 电晕放电 (corona)
  - 颗粒放电 (particle)
  - 悬浮放电 (floating)
  - 沿面放电 (surface)
  - 内部放电 (void)
- 提供REST API接口，便于集成到其他系统
- 返回识别结果和置信度百分比
- 轻量级设计，易于部署

## 三、系统架构

系统主要由以下组件构成：
- 预处理模块：将输入图像转换为标准格式 (64x64灰度图)
- 特征提取模块：使用PCA降维技术
- 分类模块：基于SVM算法的分类器
- API服务：基于FastAPI的Web服务

## 四、项目结构

```
.
├── svm_fastapi.py           # API服务主程序
├── svm_pd_model/            # 预训练模型目录
│   ├── svm_model.pkl        # SVM分类器模型
│   ├── svm_scaler.pkl       # 标准化处理器
│   └── svm_pca.pkl          # PCA降维模型
├── test_dataset/            # 测试数据集 (可选)
│   ├── corona/              # 电晕放电图像
│   ├── particle/            # 颗粒放电图像
│   ├── floating/            # 悬浮放电图像
│   ├── surface/             # 沿面放电图像
│   └── void/                # 内部放电图像
├── requirements.txt         # 依赖包列表
├── svm_request测试.py        # API测试脚本示例
├── README.md                # 本文档
└── start_service.sh         # (部署用) 启动脚本示例
```

## 五、技术细节

- **图像预处理**：将输入图像转换为64x64灰度图，然后展平为一维向量。
- **特征处理**：使用StandardScaler标准化数据，然后通过PCA进行降维。
- **分类算法**：使用SVM (支持向量机) 算法进行多类分类。
- **Web框架**：FastAPI提供高性能的异步API服务。

## 六、部署指南 (Ubuntu)

本指南详细说明如何在Ubuntu系统上部署此应用，并将其作为后台服务运行。

### 1. 系统要求

- Ubuntu 18.04 LTS 或更高版本
- Python 3.6+
- 至少2GB内存
- 至少1GB可用磁盘空间

### 2. 安装系统依赖

```bash
# 更新系统包
sudo apt update && sudo apt upgrade -y

# 安装Python及相关工具
sudo apt install -y python3 python3-pip python3-dev python3-venv
sudo apt install -y build-essential libssl-dev libffi-dev

# 安装OpenCV运行依赖
sudo apt install -y libsm6 libxext6 libxrender-dev libgl1-mesa-glx
```

### 3. 项目设置

#### a. 创建项目目录和虚拟环境

```bash
# 选择一个合适的路径创建项目目录 (例如用户主目录)
PROJECT_DIR="$HOME/pd_recognition_system" # 或者 /root/pd_recognition_system 如果必须用root
mkdir -p "$PROJECT_DIR"
cd "$PROJECT_DIR"

# 创建Python虚拟环境
python3 -m venv svm_venv

# 激活虚拟环境 (后续操作在此环境下进行)
source svm_venv/bin/activate
```

#### b. 获取项目代码

**方法一：Git克隆**
```bash
# 确保在项目目录下 ($PROJECT_DIR)
git clone <项目Git仓库URL> .
```

**方法二：手动上传**
将 `svm_fastapi.py`, `requirements.txt`, `svm_pd_model/` 目录上传到 `$PROJECT_DIR`。

#### c. 安装项目依赖

```bash
# 确保虚拟环境已激活
pip install --upgrade pip
pip install -r requirements.txt
```

> **注意: scikit-learn版本**
> 如果在安装或运行时看到 `InconsistentVersionWarning` (关于 scikit-learn 版本不匹配)，建议修改 `requirements.txt`，将 `scikit-learn` 的版本固定为训练模型时使用的版本 (例如 `scikit-learn==1.2.2`)，然后重新执行 `pip install -r requirements.txt --force-reinstall`。

### 4. 本地测试运行

在部署为服务前，先测试应用能否直接运行。

```bash
# 确保在项目目录 ($PROJECT_DIR) 且虚拟环境已激活
python svm_fastapi.py
```

如果看到 Uvicorn 成功启动并在 `http://0.0.0.0:9000` 上监听，说明基本设置正确。访问 `http://<服务器IP>:9000/docs` 确认 API 文档。按 Ctrl+C 停止。

### 5. 部署为 Systemd 服务 (推荐方案)

使用 Systemd 结合启动脚本来管理后台服务。

#### a. 创建启动脚本

在项目目录 (`$PROJECT_DIR`) 下创建 `start_service.sh` 文件：

```bash
nano start_service.sh
```

填入以下内容 (确保路径正确)：

```bash
#!/bin/bash

# 进入项目目录 (使用绝对路径)
cd "$PROJECT_DIR"

# 激活Python虚拟环境
source svm_venv/bin/activate

# 启动应用
python svm_fastapi.py
```

赋予执行权限：

```bash
chmod +x start_service.sh
```

#### b. 创建 Systemd 服务文件

```bash
# 使用你喜欢的编辑器创建服务文件
sudo nano /etc/systemd/system/pd-recognition.service
```

填入以下内容 (将 `$PROJECT_DIR` 替换为实际路径)：

```ini
[Unit]
Description=PD Recognition System API Service
After=network.target

[Service]
# 指定执行脚本的完整路径
ExecStart=/bin/bash "$PROJECT_DIR/start_service.sh"

# 指定工作目录
WorkingDirectory="$PROJECT_DIR"

# (可选) 指定运行用户和组，如果不用root运行
# User=your_username
# Group=your_group

# 失败时自动重启
Restart=on-failure
RestartSec=5s

[Install]
# 服务将在系统启动时自动启动
WantedBy=multi-user.target
```

#### c. 启动并管理服务

```bash
# 重新加载 systemd 配置
sudo systemctl daemon-reload

# 设置开机自启
sudo systemctl enable pd-recognition.service

# 启动服务
sudo systemctl start pd-recognition.service

# 查看服务状态
sudo systemctl status pd-recognition.service
```

如果状态显示 `active (running)` 则表示成功。

### 6. 配置防火墙

如果使用了 UFW 防火墙，允许 9000 端口：

```bash
sudo ufw allow 9000/tcp
sudo ufw status
```

### 7. API 测试

服务运行后，可以通过以下方式测试：

**Curl:**
```bash
curl -X POST -F "file=@./test_dataset/surface/surface57.png" http://<服务器IP>:9000/api/v1/predict
```

**Python 脚本:** (参考 `svm_request测试.py`)
```python
import requests

url = 'http://<服务器IP>:9000/api/v1/predict'
file_path = './test_dataset/surface/surface57.png' # 使用实际路径

try:
    with open(file_path, 'rb') as f:
        files = {'file': f}
        response = requests.post(url, files=files)
        response.raise_for_status() # 检查请求是否成功
        print(response.json())
except FileNotFoundError:
    print(f"Error: File not found at {file_path}")
except requests.exceptions.RequestException as e:
    print(f"Error during request: {e}")
```

## 七、系统维护

### 1. 服务管理

```bash
# 查看状态
sudo systemctl status pd-recognition.service

# 停止服务
sudo systemctl stop pd-recognition.service

# 启动服务
sudo systemctl start pd-recognition.service

# 重启服务
sudo systemctl restart pd-recognition.service

# 查看日志 (最近100条)
sudo journalctl -u pd-recognition.service -n 100 --no-pager

# 实时跟踪日志
sudo journalctl -u pd-recognition.service -f
```

### 2. 更新部署

```bash
# 进入项目目录
cd "$PROJECT_DIR"

# 更新代码 (例如使用Git)
git pull

# 激活虚拟环境
source svm_venv/bin/activate

# (如果需要) 更新依赖
pip install -r requirements.txt

# 重启服务使更改生效
sudo systemctl restart pd-recognition.service
```

## 八、常见问题与解决方案

1.  **Systemd 服务启动失败 (`status=217/USER`)**
    *   **原因**: systemd 服务文件中配置的 `User` 或 `Group` 不正确或无法访问。
    *   **解决方案**: 
        *   检查 `/etc/systemd/system/pd-recognition.service` 文件中的 `User` 和 `Group` 设置。
        *   如果使用 root 运行，可以尝试注释掉 `User` 和 `Group` 行。
        *   **强烈推荐**使用结合启动脚本的方案 (如部署指南中的方案 B)，通常能避免此问题。

2.  **Systemd 服务启动失败 (`status=2` 或 `status=1/FAILURE`)**
    *   **原因**: `ExecStart` 命令执行失败，通常是脚本路径错误、脚本本身错误或脚本没有执行权限。
    *   **解决方案**: 
        *   检查服务文件 (`/etc/systemd/system/pd-recognition.service`) 中的 `ExecStart` 和 `WorkingDirectory` 路径是否完全正确。
        *   确保 `start_service.sh` 脚本有执行权限 (`chmod +x start_service.sh`)。
        *   检查 `start_service.sh` 脚本内部的命令（如 `cd`, `source`, `python`）路径是否正确。
        *   使用 `sudo journalctl -u pd-recognition.service -n 50 --no-pager` 查看详细错误日志。

3.  **端口已被占用 (`Address already in use`)**
    *   **原因**: 尝试启动服务时，9000 端口已被其他进程占用（很可能是之前的服务实例未完全停止）。
    *   **解决方案**: 
        *   尝试停止服务：`sudo systemctl stop pd-recognition.service`。
        *   如果停止无效，查找并结束占用端口的进程：`sudo lsof -t -i:9000 | xargs --no-run-if-empty sudo kill -9`。
        *   再次尝试启动服务：`sudo systemctl start pd-recognition.service`。

4.  **scikit-learn 版本警告 (`InconsistentVersionWarning`)**
    *   **原因**: 加载的模型 (`.pkl`) 文件是用与当前环境不同版本的 scikit-learn 保存的。
    *   **解决方案**: 查看警告信息确认模型版本，修改 `requirements.txt` 文件指定该版本 (e.g., `scikit-learn==1.2.2`)，然后执行 `pip install -r requirements.txt --force-reinstall` 并重启服务。

5.  **依赖安装失败**
    *   **解决方案**: 确保已安装 `python3-dev` 和 `build-essential`。更新 pip (`pip install --upgrade pip`)。查看具体错误信息，可能需要安装特定库的系统依赖。

6.  **OpenCV 导入错误**
    *   **解决方案**: 确保已安装 OpenCV 的系统依赖 (见步骤 2.1)。尝试重新安装：`pip uninstall opencv-python && pip install opencv-python`。

## 九、性能优化建议 (可选)

1.  **使用 Gunicorn + Uvicorn Workers**: 提高并发处理能力。
    ```bash
    # 安装 gunicorn
    pip install gunicorn
    
    # 修改 start_service.sh 中的 python svm_fastapi.py 为:
    # gunicorn -w 4 -k uvicorn.workers.UvicornWorker -b 0.0.0.0:9000 svm_fastapi:app
    
    # 重启服务
    # sudo systemctl restart pd-recognition.service
    ```
    (`-w 4` 表示 4 个工作进程，可根据 CPU 核数调整；`svm_fastapi:app` 指的是 `svm_fastapi.py` 文件中的 FastAPI 应用实例 `app`)。

2.  **使用 Nginx 作为反向代理**: 提供负载均衡、HTTPS 支持和静态文件服务。
    *   安装 Nginx: `sudo apt install -y nginx`
    *   配置 Nginx (例如在 `/etc/nginx/sites-available/pd-recognition`) 将请求代理到 `http://127.0.0.1:9000`。

## 十、注意事项

- 服务默认在 9000 端口运行。如果需要修改，请同时更新 `svm_fastapi.py` 代码和防火墙规则。
- 上传的图像会临时保存为 `temp_image.jpg`，确保程序有写入权限。
- 为获得最佳识别效果，建议上传清晰的局放图像。
- 生产环境强烈建议配置 HTTPS (通常通过 Nginx 反向代理实现)。
- 定期备份模型文件 (`svm_pd_model/` 目录)。
- 考虑日志轮转以防止日志文件过大。 