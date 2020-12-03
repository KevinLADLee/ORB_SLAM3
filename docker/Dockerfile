ARG ROS_VERSION='noetic'

FROM osrf/ros:$ROS_VERSION-desktop-focal

RUN sed -i "s@http://.*archive.ubuntu.com@http://mirrors.huaweicloud.com@g" /etc/apt/sources.list && \
    sed -i "s@http://.*security.ubuntu.com@http://mirrors.huaweicloud.com@g" /etc/apt/sources.list && \
    sed -i "s@http://packages.ros.org/ros/ubuntu@https://mirror.tuna.tsinghua.edu.cn/ros/ubuntu/@g" /etc/apt/sources.list.d/ros1-latest.list


ARG ROS_VERSION
RUN apt-get update && \
    if [ "$ROS_VERSION" = "noetic" ] ; then export PYTHON_SUFFIX=3 ; else export PYTHON_SUFFIX="" ; fi && \
    apt-get install -y --no-install-recommends \
        vim git curl wget unzip sudo \
        ros-$ROS_VERSION-pcl-ros \
        ros-$ROS_VERSION-pcl-conversions \
        ros-$ROS_VERSION-cv-bridge \
        python$PYTHON_SUFFIX-osrf-pycommon \
        python$PYTHON_SUFFIX-catkin-tools \
        python3-pip  \
    && rm -rf /var/lib/apt/lists/*

RUN if [ "x$(nproc)" = "x1" ] ; then export USE_PROC=1 ; else export USE_PROC=$(($(nproc)-1)) ; fi && \
    apt-get update && \
    apt-get install -y --no-install-recommends \
        libglew-dev \
        libgl1-mesa-dev && \
    rm -rf /var/lib/apt/lists/* && \
    mkdir -p /opt/orb-slam/ && \
    cd /opt/orb-slam/ && \ 
    git clone https://github.com/stevenlovegrove/Pangolin.git --depth=1 && \
    cd Pangolin && \
    mkdir build && cd build && cmake .. -DCMAKE_BUILD_TYPE=Release && make -j${USE_PROC} install 

COPY ./ /opt/orb-slam/ORB_SLAM3    

WORKDIR /opt/orb-slam/ORB_SLAM3 

RUN bash ./build.sh && \
    bash -c "source /opt/ros/noetic/setup.bash; export ROS_PACKAGE_PATH=\${ROS_PACKAGE_PATH}:/opt/orb-slam/ORB_SLAM3/Examples/ROS; ./build_ros.sh"

# RUN if [ "x$(nproc)" = "x1" ] ; then export USE_PROC=1 ; else export USE_PROC=$(($(nproc)-1)) ; fi && \
#     mkdir -p /opt/lio_sam_ws/src && \
#     mv /opt/LIO-SAM /opt/lio_sam_ws/src && \
#     cd /opt/lio_sam_ws && \
#     bash -c "source /opt/ros/$ROS_VERSION/setup.bash; catkin_make -j${USE_PROC}" && \
#     sed -i '/exec/i\source /opt/lio_sam_ws/devel/setup.bash' /ros_entrypoint.sh