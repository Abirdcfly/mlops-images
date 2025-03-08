###############################################################
# Dockerfile that builds 'yanwk/comfyui-boot:cu121'
# A runtime environment for https://github.com/comfyanonymous/ComfyUI
# Using CUDA 12.1 & Python 3.11
################################################################################
ARG PIP_SOURCE=https://pypi.mirrors.ustc.edu.cn/simple
ARG MODEL_TYPE
ARG MODEL_VERSION

FROM nvidia/cuda:12.1.0-runtime-ubuntu22.04

ARG MODEL_TYPE
ARG MODEL_VERSION

# Set environment variables
ENV TZ=Asia/Shanghai
ENV LANG=zh_CN.UTF-8
ENV DEBIAN_FRONTEND=noninteractive
RUN ln -snf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone

# Install system dependencies
RUN --mount=type=cache,target=/var/cache/apt --mount=type=cache,target=/var/lib/apt/lists \
    apt-get update && apt-get install -y --no-install-recommends \
    software-properties-common g++ make cmake curl libopencv-dev ffmpeg libx264-dev libx265-dev \
    libglib2.0-0 git aria2 mesa-utils fonts-noto-cjk fonts-noto-color-emoji openssh-server
RUN --mount=type=cache,target=/var/cache/apt --mount=type=cache,target=/var/lib/apt/lists \
    add-apt-repository ppa:deadsnakes/ppa
RUN --mount=type=cache,target=/var/cache/apt --mount=type=cache,target=/var/lib/apt/lists \
    apt-get update && apt-get install -y --no-install-recommends \
    python3.11 python3.11-dev python3.11-venv python3.11-distutils
RUN --mount=type=cache,target=/var/cache/apt --mount=type=cache,target=/var/lib/apt/lists \
    curl -sS https://bootstrap.pypa.io/get-pip.py | python3.11
RUN --mount=type=cache,target=/var/cache/apt --mount=type=cache,target=/var/lib/apt/lists \
    update-alternatives --install /usr/bin/python3 python3 /usr/bin/python3.11 100
RUN --mount=type=cache,target=/var/cache/apt --mount=type=cache,target=/var/lib/apt/lists \
    rm -rf /var/lib/apt/lists/*

# Set up Python environment
RUN --mount=type=cache,target=/root/.cache/pip \
    pip3.11 install --upgrade pip wheel setuptools
RUN --mount=type=cache,target=/root/.cache/pip \
    pip3.11 install 'numpy<2.0' jupyterlab jupyterlab-language-pack-zh-CN jupyterlab_pygments tensorboard
RUN --mount=type=cache,target=/root/.cache/pip \
    pip3.11 install xformers
RUN --mount=type=cache,target=/root/.cache/pip \
    pip3.11 install torchvision
RUN --mount=type=cache,target=/root/.cache/pip \
    pip3.11 install torchaudio

# Install ComfyUI and other Python dependencies
RUN --mount=type=cache,target=/root/.cache/pip \
    pip3.11 install -r https://raw.githubusercontent.com/comfyanonymous/ComfyUI/master/requirements.txt \
    -r https://raw.githubusercontent.com/crystian/ComfyUI-Crystools/main/requirements.txt \
    -r https://raw.githubusercontent.com/cubiq/ComfyUI_essentials/main/requirements.txt \
    -r https://raw.githubusercontent.com/Fannovel16/comfyui_controlnet_aux/main/requirements.txt \
    -r https://raw.githubusercontent.com/jags111/efficiency-nodes-comfyui/main/requirements.txt \
    -r https://raw.githubusercontent.com/ltdrdata/ComfyUI-Impact-Pack/Main/requirements.txt \
    -r https://raw.githubusercontent.com/ltdrdata/ComfyUI-Impact-Subpack/main/requirements.txt \
    -r https://raw.githubusercontent.com/ltdrdata/ComfyUI-Inspire-Pack/main/requirements.txt \
    -r https://raw.githubusercontent.com/ltdrdata/ComfyUI-Manager/main/requirements.txt

# Install additional Python packages
RUN --mount=type=cache,target=/root/.cache/pip \
    pip3.11 install cmake onnxruntime insightface color_matcher onnxruntime-gpu torch numpy einops \
    opencv-contrib-python kornia scipy Pillow torchvision tqdm pandas numexpr 'pillow>=10.3.0' \
    matplotlib huggingface_hub mss opencv-python 'qrcode[pil]' requirements-parser rembg \
    imageio_ffmpeg rich rich_argparse ninja PyGithub requests 'nerfacc>=0.5.3' slangtorch \
    'diffusers>=0.26.1' 'transformers>=4.36.2' safetensors open_clip_torch torchmetrics \
    pytorch_msssim pytorch-lightning trimesh plyfile pygltflib xatlas pymeshlab gpytoolbox \
    PyMCubes torchtyping jaxtyping packaging OmegaConf pyhocon segment_anything timm addict \
    yapf cupy-cuda12x accelerate compel lark torchdiffeq fairscale python-ffmpeg boost dlib

# Fix ONNX Runtime and MediaPipe
RUN --mount=type=cache,target=/root/.cache/pip \
    pip3.11 uninstall --yes onnxruntime-gpu && \
    pip3.11 install --no-cache-dir onnxruntime-gpu --index-url https://aiinfra.pkgs.visualstudio.com/PublicPackages/_packaging/onnxruntime-cuda-12/pypi/simple/ \
    --extra-index-url https://pypi.org/simple && pip3.11 install mediapipe

# Set up library paths
ENV LD_LIBRARY_PATH="${LD_LIBRARY_PATH}:/usr/local/lib64/python3.11/site-packages/torch/lib:/usr/local/lib/python3.11/site-packages/nvidia/cuda_cupti/lib:/usr/local/lib/python3.11/site-packages/nvidia/cuda_runtime/lib:/usr/local/lib/python3.11/site-packages/nvidia/cudnn/lib:/usr/local/lib/python3.11/site-packages/nvidia/cufft/lib:/usr/local/lib/python3.11/site-packages/nvidia/cublas/lib:/usr/local/lib/python3.11/site-packages/nvidia/cuda_nvrtc/lib:/usr/local/lib/python3.11/site-packages/nvidia/curand/lib:/usr/local/lib/python3.11/site-packages/nvidia/cusolver/lib:/usr/local/lib/python3.11/site-packages/nvidia/cusparse/lib:/usr/local/lib/python3.11/site-packages/nvidia/nccl/lib:/usr/local/lib/python3.11/site-packages/nvidia/nvjitlink/lib:/usr/local/lib/python3.11/site-packages/nvidia/nvtx/lib"

# Set up SSH and other configurations
RUN mkdir /var/run/sshd && \
    echo "root:123456" | chpasswd && \
    sed -i 's/#PermitRootLogin prohibit-password/PermitRootLogin yes/' /etc/ssh/sshd_config && \
    echo "PasswordAuthentication yes" >> /etc/ssh/sshd_config && \
    echo "source /etc/profile" >> /root/.bashrc && \
    echo "source /etc/matrixdc-motd" >> /root/.bashrc

# Create necessary directories and copy files
RUN mkdir -p /init/ /runner-scripts/ /comfyui /etc/supervisord
COPY ./init/ /init/
COPY ./runner-scripts/ /runner-scripts/
RUN chmod 755 /init/boot/*.sh && chmod 755 /init/bin/* && \
    chmod 755 /runner-scripts/*.sh

# Set up ComfyUI
ENV HF_ENDPOINT=https://hf-mirror.com
WORKDIR /comfyui
RUN bash /runner-scripts/build-comfy.sh ${MODEL_TYPE} app_tag_001

COPY ./${MODEL_VERSION} /comfyui/ComfyUI/models/checkpoints/${MODEL_VERSION}

# Set up supervisord for ComfyUI
RUN echo "[program:comfyui]" >> /etc/supervisord/supervisor-other.ini && \
    echo "command=python3 /comfyui/ComfyUI/main.py --listen --port 3000" >> /etc/supervisord/supervisor-other.ini && \
    echo "autostart=true" >> /etc/supervisord/supervisor-other.ini && \
    echo "autorestart=true" >> /etc/supervisord/supervisor-other.ini && \
    echo "stderr_logfile=/var/log/comfyui.err.log" >> /etc/supervisord/supervisor-other.ini && \
    echo "stdout_logfile=/var/log/comfyui.out.log" >> /etc/supervisord/supervisor-other.ini

# Expose ports
EXPOSE 22 3000 8888 6006

# Set the entrypoint
CMD ["bash", "/init/boot/boot.sh"]
