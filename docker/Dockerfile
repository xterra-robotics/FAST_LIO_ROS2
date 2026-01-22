# Author: TaeYoung Kim
# email: tyoung96@yonsei.ac.kr
FROM dustynv/ros:humble-ros-base-l4t-r36.2.0

# Set non-interactive installation
ENV DEBIAN_FRONTEND=noninteractive

# Fix the expired ROS GPG key FIRST
RUN apt-get update 2>/dev/null || true && \
    apt-get install -y curl gnupg2 || true && \
    curl -sSL https://raw.githubusercontent.com/ros/rosdistro/master/ros.key -o /usr/share/keyrings/ros-archive-keyring.gpg && \
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/ros-archive-keyring.gpg] http://packages.ros.org/ros2/ubuntu $(. /etc/os-release && echo $UBUNTU_CODENAME) main" | tee /etc/apt/sources.list.d/ros2.list > /dev/null

# Clean and update
RUN apt-get clean && \
    rm -rf /var/lib/apt/lists/* && \
    apt-get update

# Install base utilities
RUN apt-get install -y \
    wget \
    curl \
    git \
    usbutils \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install build tools
RUN apt-get update && apt-get install -y \
    cmake \
    build-essential \
    python3-pip \
    python3-colcon-common-extensions \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install ROS packages WITHOUT cv-bridge
RUN apt-get update && apt-get install -y \
    ros-humble-image-transport \
    ros-humble-pcl-conversions \
    ros-humble-pcl-ros \
    ros-humble-tf2 \
    ros-humble-tf2-ros \
    ros-humble-sensor-msgs \
    ros-humble-geometry-msgs \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install libraries (skip OpenCV - already in base image)
RUN apt-get update && apt-get install -y \
    libatlas-base-dev \
    libeigen3-dev \
    libpcl-dev \
    libgoogle-glog-dev \
    libsuitesparse-dev \
    libglew-dev \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Install Livox SDK
WORKDIR /root/
RUN git clone https://github.com/Livox-SDK/Livox-SDK2.git && \
    cd Livox-SDK2 && \
    mkdir build && cd build && \
    cmake .. && make -j$(nproc) && make install && \
    cd /root && rm -rf Livox-SDK2

# Build cv-bridge from source (skip tests to avoid ament_lint dependency)
WORKDIR /root/ros2_ws/src
RUN git clone --branch humble https://github.com/ros-perception/vision_opencv.git && \
    cd /root/ros2_ws && \
    . /opt/ros/humble/setup.sh && \
    colcon build --packages-select cv_bridge \
        --cmake-args -DCMAKE_BUILD_TYPE=Release -DBUILD_TESTING=OFF && \
    rm -rf build log

# Additional Packages
RUN apt-get update && apt-get install -y \
    ros-humble-rmw-cyclonedds-cpp \
    ros-humble-ament-cmake-auto \
    ros-humble-rosidl-default-generators \
    ros-humble-ament-lint-auto \ 
    ros-humble-ament-lint-common \
    ros-humble-nav-msgs \
    ros-humble-std-srvs \
    ros-humble-visualization-msgs \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Source ROS 2 
RUN echo "source /opt/ros/humble/setup.bash" >> /root/.bashrc && \
    echo "source /root/ros2_ws/install/setup.bash" >> /root/.bashrc

WORKDIR /root/ros2_ws

# Load ROS environment at each run
COPY ./ros_entrypoint.sh /
RUN chmod 755 /ros_entrypoint.sh
ENTRYPOINT ["/ros_entrypoint.sh"]

CMD ["bash"]
