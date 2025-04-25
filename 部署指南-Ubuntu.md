# 局放图像识别系统 - Ubuntu部署指南

本文档提供在Ubuntu系统上部署局放图像识别系统的详细步骤。

## 系统要求

- Ubuntu 18.04 LTS 或更高版本
- Python 3.6+
- 至少2GB内存
- 至少1GB可用磁盘空间

## 部署步骤

### 1. 安装系统依赖

```bash
# 更新系统包
sudo apt update
sudo apt upgrade -y

# 安装Python及相关工具
sudo apt install -y python3 python3-pip python3-dev
sudo apt install -y python3-venv
sudo apt install -y build-essential libssl-dev libffi-dev

# 安装OpenCV依赖
sudo apt install -y libsm6 libxext6 libxrender-dev libgl1-mesa-glx
```

### 2. 创建项目目录

```bash
# 创建项目目录
mkdir -p ~/pd_recognition_system
cd ~/pd_recognition_system
```

### 3. 设置Python虚拟环境

```bash
# 创建虚拟环境
python3 -m venv venv

# 激活虚拟环境
source venv/bin/activate
```

### 4. 获取项目代码

方法一：克隆Git仓库（如果使用Git）

```bash
git clone <项目Git仓库URL> .
```

方法二：手动上传项目文件

使用SCP或SFTP等工具将项目文件上传到Ubuntu服务器的`~/pd_recognition_system`目录下。确保上传以下内容：

- `svm_fastapi.py`
- `requirements.txt`
- `svm_pd_model/` 目录及其中的所有文件
- `test_dataset/` 目录（可选，仅用于测试）

### 5. 安装项目依赖

```bash
# 确保虚拟环境已激活
pip install --upgrade pip
pip install -r requirements.txt
```

### 6. 测试服务

```bash
# 运行服务
python svm_fastapi.py
```

访问 `http://<服务器IP>:9000/docs` 检查API文档界面是否正常加载。

### 7. 配置系统服务（使服务在后台运行）

创建systemd服务文件：

```bash
sudo nano /etc/systemd/system/pd-recognition.service
```

添加以下内容：

```
[Unit]
Description=PD Recognition System API Service
After=network.target

[Service]
User=<你的用户名>
Group=<你的用户组>
WorkingDirectory=/home/<你的用户名>/pd_recognition_system
Environment="PATH=/home/<你的用户名>/pd_recognition_system/venv/bin"
ExecStart=/home/<你的用户名>/pd_recognition_system/venv/bin/python svm_fastapi.py

[Install]
WantedBy=multi-user.target
```

注意：替换 `<你的用户名>` 和 `<你的用户组>` 为实际值。可以通过 `whoami` 和 `groups` 命令获取。

实测例子-以香橙派开发版安装ubuntu系统的系统服务配置如下(已测试成功)：
我是root(管理员)运行方式。
添加内容如下：
```

[Unit]
Description=PD Recognition System API Service
After=network.target

[Service]
User=root
Group=root
WorkingDirectory=/root/pd_recognition_system
Environment="PATH=/root/pd_recognition_system/svm_venv/bin"
ExecStart=/root/pd_recognition_system/svm_venv/bin/python svm_fastapi.py

[Install]
WantedBy=multi-user.target

```


启用和运行服务：

```bash
sudo systemctl daemon-reload
sudo systemctl enable pd-recognition.service
sudo systemctl start pd-recognition.service
```

检查服务状态：

```bash
sudo systemctl status pd-recognition.service
```

### 8. 配置防火墙（如需对外提供服务）

```bash
# 允许9000端口通过防火墙
sudo ufw allow 9000/tcp
sudo ufw status
```

## API测试

### 使用curl测试API

```bash
curl -X POST -F "file=@./test_dataset/corona/corona111.png" http://localhost:9000/api/v1/predict
```

### 使用Python脚本测试

创建测试脚本 `test_api.py`：

```python
import requests

url = 'http://localhost:9000/api/v1/predict'
file_path = './test_dataset/surface/surface57.png'  # 替换为实际图像路径
files = {'file': open(file_path, 'rb')}

response = requests.post(url, files=files)
print(response.json())
```

运行测试：

```bash
python test_api.py
```

## 系统维护

### 查看日志

```bash
sudo journalctl -u pd-recognition.service
```

### 重启服务

```bash
sudo systemctl restart pd-recognition.service
```

### 停止服务

```bash
sudo systemctl stop pd-recognition.service
```

## 可能遇到的问题及解决方案

### 1. 依赖安装失败

问题：安装某些Python依赖包时出错。

解决方案：
```bash
# 安装Python开发包
sudo apt install -y python3-dev

# 安装编译工具
sudo apt install -y build-essential
```

### 2. OpenCV导入错误

问题：导入OpenCV库时出现错误。

解决方案：
```bash
# 安装OpenCV系统依赖
sudo apt install -y libsm6 libxext6 libxrender-dev libgl1-mesa-glx
```

### 3. 服务无法访问

问题：无法从外部访问API服务。

解决方案：
1. 检查防火墙设置
```bash
sudo ufw status
sudo ufw allow 9000/tcp
```

2. 确保FastAPI服务绑定到`0.0.0.0`而不是`127.0.0.1`
3. 检查云服务提供商的安全组/网络ACL设置

### 4. 内存不足

问题：服务运行一段时间后崩溃，日志显示内存不足。

解决方案：
1. 增加虚拟机内存
2. 配置swap空间
```bash
sudo fallocate -l 2G /swapfile
sudo chmod 600 /swapfile
sudo mkswap /swapfile
sudo swapon /swapfile
echo '/swapfile none swap sw 0 0' | sudo tee -a /etc/fstab
```

## 性能优化建议

1. 考虑使用Gunicorn作为WSGI服务器与Uvicorn配合使用，提高并发处理能力

```bash
pip install gunicorn
gunicorn -w 4 -k uvicorn.workers.UvicornWorker app:app
```

2. 使用Nginx作为反向代理，提供更好的负载均衡和安全性

```bash
sudo apt install -y nginx
```

配置Nginx：
```
server {
    listen 80;
    server_name your_domain.com;

    location / {
        proxy_pass http://127.0.0.1:9000;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
    }
}
``` 