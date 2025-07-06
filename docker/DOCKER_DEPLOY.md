# ComfyUI Docker 部署详细文档

## 1. Dockerfile 编写说明

推荐使用如下 Dockerfile（适用于国内环境、NVIDIA GPU）：

```dockerfile
FROM python:3.12-slim

# 写入阿里云Debian源，适配slim镜像无sources.list情况
RUN echo "deb https://mirrors.aliyun.com/debian bookworm main contrib non-free non-free-firmware\ndeb https://mirrors.aliyun.com/debian-security bookworm-security main contrib non-free non-free-firmware\ndeb https://mirrors.aliyun.com/debian bookworm-updates main contrib non-free non-free-firmware\ndeb https://mirrors.aliyun.com/debian bookworm-backports main contrib non-free non-free-firmware" > /etc/apt/sources.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
        git \
        ffmpeg \
        libgl1 \
        libglib2.0-0 \
    && rm -rf /var/lib/apt/lists/*

WORKDIR /app
COPY . /app

RUN pip install --upgrade pip
RUN pip install torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu128
RUN pip install --no-cache-dir -r requirements.txt --extra-index-url https://download.pytorch.org/whl/cu128

EXPOSE 8188
CMD ["python", "main.py"]
```

## 2. 镜像打包（构建）

在 ComfyUI 项目根目录下执行：
```powershell
docker build -t comfyui:latest .
```

## 3. 镜像导出与离线部署（可选）

如需在无外网服务器部署：
```powershell
docker save -o comfyui.tar comfyui:latest
# 拷贝 comfyui.tar 到目标服务器
# 目标服务器导入：
docker load -i comfyui.tar
```

## 4. 启动容器（含文件挂载）

假设本地有如下目录：
- D:/comfyui/models      （模型文件）
- D:/comfyui/input       （输入图片等）
- D:/comfyui/output      （输出图片等）

启动命令如下（含GPU支持）：
```powershell
docker run --gpus all -d -p 8188:8188 \
  -v /data/comfyui/models:/app/models \
  -v /data/comfyui/input:/app/input \
  -v /data/comfyui/output:/app/output \
  --name comfyui comfyui:latest
```

如需挂载更多目录（如 custom_nodes、配置文件等），可继续添加 -v 参数。

## 5. 访问服务

浏览器访问：http://localhost:8188

## 6. 常用运维命令

- 查看日志：
  ```powershell
  docker logs -f comfyui
  ```
- 停止容器：
  ```powershell
  docker stop comfyui
  ```
- 删除容器：
  ```powershell
  docker rm comfyui
  ```
- 删除镜像：
  ```powershell
  docker rmi comfyui:latest
  ```

## 7. 使用 docker-compose 部署（推荐Linux环境）

在项目根目录新建 `docker-compose.yml`，内容如下：

```yaml
version: '3.8'
services:
  comfyui:
    image: comfyui:latest
    container_name: comfyui
    restart: unless-stopped
    ports:
      - "8188:8188"
    deploy:
      resources:
        reservations:
          devices:
            - driver: nvidia
              count: all
              capabilities: [gpu]
    volumes:
      - C:/Users/18271/models:/app/models
      - C:/Users/18271/input:/app/input
      - C:/Users/18271/output:/app/output
      # windwows 下可使用 \\ 代替 /
    # volumes:
    #   - /data/comfyui/models:/app/models
    #   - /data/comfyui/input:/app/input
    #   - /data/comfyui/output:/app/output
      # 如有需要可继续挂载 custom_nodes、配置文件等
```

启动服务：
```bash
docker compose up -d
```

如需关闭服务：
```bash
docker compose down
```

如需重启服务：
```bash
docker compose restart
```

---
如需自定义端口、环境变量或其他挂载，请在 volumes/ports 部分自行调整。
如需 CPU 版本或其他特殊需求，请补充说明。

+