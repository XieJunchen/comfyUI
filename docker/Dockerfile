# 基于官方Python 3.10.11镜像
FROM python:3.10.11-slim
# FROM python:3.12-slim

# 设置工作目录
WORKDIR /app

# 写入阿里云Debian源，适配slim镜像无sources.list情况
RUN echo "deb https://mirrors.aliyun.com/debian bookworm main contrib non-free non-free-firmware\ndeb https://mirrors.aliyun.com/debian-security bookworm-security main contrib non-free non-free-firmware\ndeb https://mirrors.aliyun.com/debian bookworm-updates main contrib non-free non-free-firmware\ndeb https://mirrors.aliyun.com/debian bookworm-backports main contrib non-free non-free-firmware" > /etc/apt/sources.list && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
        gcc \
        python3-dev \
        libffi-dev \
        libssl-dev \
        build-essential \
        git \
        ffmpeg \
        libgl1 \
        libglib2.0-0 \
    && rm -rf /var/lib/apt/lists/*

# 拷贝项目文件
COPY . /app

# 先升级pip
RUN pip install --upgrade pip

# 安装CUDA 12.8版本PyTorch，增加重试和超时时间，网络不佳时可多次自动尝试
RUN pip install --retries 10 --timeout 120 torch torchvision torchaudio --extra-index-url https://download.pytorch.org/whl/cu128

# 安装其余依赖（不再通过requirements.txt安装torch/torchvision/torchaudio）
RUN pip install --no-cache-dir -r requirements.txt --extra-index-url https://download.pytorch.org/whl/cu128

# 安装comfyui_manager依赖（假设路径为custom_nodes/ComfyUI-Manager/requirements.txt）
RUN if [ -f custom_nodes/ComfyUI-Manager/requirements.txt ]; then \
    pip install --no-cache-dir -r custom_nodes/ComfyUI-Manager/requirements.txt; \
fi

# 暴露端口
EXPOSE 8188

# 启动命令
CMD ["python", "main.py"]
