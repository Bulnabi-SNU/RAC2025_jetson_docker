FROM dustynv/l4t-pytorch:r36.4.0

ARG DEBIAN_FRONTEND=noninteractive
ENV TZ=UTC LANG=en_US.UTF-8 LC_ALL=en_US.UTF-8 TORCHVISION_DISABLE_OPERATOR_PRELU=1

RUN apt-get update && apt-get install -y --no-install-recommends \
    software-properties-common curl gnupg2 lsb-release locales ca-certificates && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

RUN add-apt-repository universe

RUN curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key \
        -o /usr/share/keyrings/ros-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] \
         http://packages.ros.org/ros2/ubuntu $(lsb_release -cs) main" \
         > /etc/apt/sources.list.d/ros2.list

RUN apt-get update

RUN apt-get install -y --no-install-recommends \
    build-essential cmake ninja-build pkg-config git wget vim nano sudo v4l-utils && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

RUN apt-get update && apt-get install -y --no-install-recommends \
    libgeographic-dev geographiclib-tools && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

RUN apt-get update && apt-get install -y --no-install-recommends \
    python3-colcon-common-extensions && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

RUN apt-get update && apt-get install -y --no-install-recommends \
    ros-humble-ros-base && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

RUN apt-get update && apt-get install -y --no-install-recommends \
    ros-humble-mavros ros-humble-mavros-extras && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

RUN apt-get update && apt-get install -y --no-install-recommends \
    ros-humble-vision-msgs ros-humble-image-transport && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

RUN locale-gen en_US.UTF-8

RUN python3 -m pip install --upgrade pip && \
    python3 -m pip install --no-cache-dir numpy==1.26.4 && \
    python3 -m pip install --no-cache-dir ultralytics==8.3.159 && \
    python3 -m pip install --no-cache-dir mavsdk && \
    python3 -m pip install --no-cache-dir 'grpcio>=1.54,<2.0' && \
    python3 -m pip install --no-cache-dir apriltag && \
    python3 -m pip install --no-cache-dir pyserial==3.5

RUN python3 -m pip install --no-cache-dir \
    --extra-index-url https://pypi.jetson-ai-lab.dev/jp6/cu126 \
    opencv-python

RUN CUDA_VISIBLE_DEVICES="" yolo checks || true

RUN echo "source /opt/ros/humble/setup.bash" >> /root/.bashrc && \
    git config --global --add safe.directory '*'

WORKDIR /ros2_ws
RUN mkdir -p src

COPY --chmod=755 <<'EOF' /entrypoint.sh
#!/usr/bin/env bash
set -e
source /opt/ros/humble/setup.bash
cd /ros2_ws
if [[ -d src && $(ls -A src) ]]; then
  JOBS=${COLCON_JOBS:-2}
  colcon build --symlink-install --parallel-workers "${JOBS}"
  source /ros2_ws/install/setup.bash
fi
exec "$@"
EOF
    
RUN apt-get update && apt-get install -y --no-install-recommends \
        git cmake build-essential libasio-dev libtinyxml2-dev && \
    git clone --recursive https://github.com/eProsima/Micro-XRCE-DDS-Agent.git /tmp/Micro-XRCE-DDS-Agent && \
    mkdir -p /tmp/Micro-XRCE-DDS-Agent/build && \
    cd /tmp/Micro-XRCE-DDS-Agent/build && \
    cmake .. -DCMAKE_BUILD_TYPE=Release && \
    make -j$(nproc) && \
    make install && \
    ldconfig && \
    cd / && rm -rf /tmp/Micro-XRCE-DDS-Agent && \
    apt-get clean && rm -rf /var/lib/apt/lists/*

ENTRYPOINT ["/entrypoint.sh"]
CMD ["bash"]

