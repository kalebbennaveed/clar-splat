# FROM ubuntu:22.04
FROM nvidia/cuda:11.8.0-devel-ubuntu22.04

## necessary tooling for px4
RUN apt-get update 
# Update and install Python 3.10 + dependencies
RUN apt-get update && apt-get install -y \
    python3.10 \
    python3.10-dev \
    python3.10-venv \
    python3-pip \
    && rm -rf /var/lib/apt/lists/*


# Ensure python3 points to python3.10
RUN ln -sf /usr/bin/python3.10 /usr/bin/python3

# Upgrade pip and install necessary Python packages
RUN python3 -m pip install --upgrade pip \
    && python3 -m pip install setuptools==58.2.0 empy pyros-genmsg

# Set default shell
SHELL ["/bin/bash", "-c"]

CMD ["python3"]

# Downgrade setuptools to a compatible version
RUN python3 -m pip install setuptools==58.2.0
RUN python3 -m pip install --user -U empy pyros-genmsg

RUN apt-get update && apt-get install -y git vim


RUN apt install -y software-properties-common
RUN add-apt-repository universe
RUN apt update && apt install curl -y
RUN curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg
RUN echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null
RUN apt update

ARG DEBIAN_FRONTEND=noninteractive

RUN apt install -y ros-humble-ros-base ros-dev-tools
RUN echo "source /opt/ros/humble/setup.bash" >> ~/.bashrc

RUN apt-get install -y libboost-all-dev ros-humble-diagnostic-updater
RUN apt install -y tmux iputils-ping

# **Fix NumPy Compatibility Issue**
RUN python3 -m pip install numpy==1.26.4

WORKDIR /home/colcon_ws
# RUN echo "source /home/colcon_ws/install/setup.bash" >> ~/.bashrc

# Install PyTorch and dependencies
RUN python3 -m pip install torch==2.1.2+cu118 torchvision==0.16.2+cu118 --extra-index-url https://download.pytorch.org/whl/cu118

# Set CUDA architecture for tiny-cuda-nn
ENV TCNN_CUDA_ARCHITECTURES=80  

# # Install TinyCUDANN dependency
RUN python3 -m pip install ninja git+https://github.com/NVlabs/tiny-cuda-nn/#subdirectory=bindings/torch

# Install pyliblzfse separately
RUN pip install --no-cache-dir pyliblzfse

WORKDIR /home/colcon_ws

RUN apt update
RUN apt install -y libgl1

ARG CUDA_ARCHITECTURES="90;89;86;80;75;70;61"
RUN pip install cv_bridge
RUN pip uninstall gsplat -y

RUN export TORCH_CUDA_ARCH_LIST="$(echo "$CUDA_ARCHITECTURES" | tr ';' '\n' | awk '$0 > 70 {print substr($0,1,1)"."substr($0,2)}' | tr '\n' ' ' | sed 's/ $//')" && \
    export MAX_JOBS=4 && \
    pip install --no-cache-dir git+https://github.com/nerfstudio-project/gsplat.git@c7b0a383657307a13dff56cb2f832e3ab7f029fd

RUN pip uninstall -y blinker || true
RUN pip install --ignore-installed blinker
RUN pip install fire==0.6.0  # Use an older, more stable version

RUN pip install gsplat==0.1.11 
RUN pip install nerfstudio==1.1.5
RUN pip install Clarabel==0.10.0 # for solving QP
RUN pip install viser==0.2.23

# Install NerfBridge
RUN git clone https://github.com/javieryu/nerf_bridge.git
WORKDIR /root/nerf_bridge

# Install NerfBridge dependencies
RUN pip install -e .
