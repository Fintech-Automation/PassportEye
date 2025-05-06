# 🛂 PassportEye Deployment & Management Guide

本指南介绍如何在 AWS EC2 上部署 PassportEye + FastAPI 项目，并实现服务开机自启、代码更新与日志查看。

---

## ✅ 1. 安装与环境准备
### Ubuntu Server 22.04 LTS (HVM), SSD Volume Type


```bash
# 1. 创建一个项目目录
mkdir ~/passporteye-api
cd ~/passporteye-api

# 2. 创建 Python 虚拟环境
python3 -m venv venv
source venv/bin/activate

# 3. 安装依赖
pip install passporteye fastapi uvicorn python-multipart pillow opencv-python-headless

# 4. 创建 main.py（此时你就在 passporteye-api 目录下）
nano main.py
```


---

## ✅ 2. 测试运行 API

```bash
# 1. 进入项目目录
cd ~/passporteye-api
# 2. 激活虚拟环境
source venv/bin/activate
# 3. 直接运行
uvicorn main:app --host 0.0.0.0 --port 8000
```

浏览器打开：

```
http://<EC2_PUBLIC_IP>:8000/docs
```

---

## ⚙️ 3. 设置 systemd 实现开机自启

### 3.1 创建服务文件

```bash
sudo nano /etc/systemd/system/passporteye.service
```

粘贴以下内容（注意路径根据实际调整）：

```ini
[Unit]
Description=PassportEye FastAPI Service
After=network.target

[Service]
User=ubuntu
WorkingDirectory=/home/ubuntu/passporteye-api
ExecStart=/home/ubuntu/passporteye-api/venv/bin/uvicorn main:app --host 0.0.0.0 --port 8000
Restart=always
RestartSec=5

[Install]
WantedBy=multi-user.target
```

### 3.2 启用并启动服务

```bash
sudo systemctl daemon-reexec
sudo systemctl daemon-reload
sudo systemctl enable passporteye.service
sudo systemctl start passporteye.service
```

---

## 🔁 4. 更新代码并重新加载服务

每次修改了 `main.py` 或配置后，运行：

```bash
sudo systemctl restart passporteye.service
```

---

## 🧾 5. 查看运行状态和日志

### 查看服务状态：

```bash
sudo systemctl status passporteye.service
```

### 查看最新运行日志：

```bash
journalctl -u passporteye.service -n 50 --no-pager
```

---

## 🧪 6. API 调用测试

POST 上传接口：

```
POST http://<EC2_PUBLIC_IP>:8000/extract-mrz
Content-Type: multipart/form-data
Body: file = passport.jpg
```

或访问：

```
http://<EC2_PUBLIC_IP>:8000/docs
```

---

## ✅ 7. 项目结构建议

```
~/passporteye-api/
├── main.py              # FastAPI 入口
├── venv/                # 虚拟环境
├── __pycache__/
```
