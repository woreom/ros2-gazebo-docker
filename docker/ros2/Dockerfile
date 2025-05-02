# https://hub.docker.com/_/ros
#FROM ros:galactic

# https://github.com/osrf/docker_images/blob/master/README.md#ros---1
# https://github.com/osrf/docker_images/blob/master/ros/galactic/ubuntu/focal/desktop/Dockerfile
FROM osrf/ros:galactic-desktop

ARG user_id=1000
ARG ros_ws=/home/dockeruser/dev_ws

ENV DEBIAN_FRONTEND=noninteractive

# Install packages.
RUN apt-get update && apt-get install -y \
    sudo \
    ros-${ROS_DISTRO}-gazebo-* \
    ros-${ROS_DISTRO}-ros2-* \
    ros-galactic-twist-mux \
    python-is-python3 \
    nano \
    less \
    xterm \
    ros-${ROS_DISTRO}-nav* \
    ros-${ROS_DISTRO}-nav2-*\
    ros-${ROS_DISTRO}-slam-* 
#    && rm -rf /var/lib/apt/lists/*

# Set root password
RUN echo 'root:1' | chpasswd

# Switch to a non-root user and add to sudo group.
RUN useradd -m --uid ${user_id} dockeruser \
    && echo 'dockeruser:1' | chpasswd \
    && adduser dockeruser sudo
USER dockeruser

# Working directory
WORKDIR /home/dockeruser

# ENVs
ENV HOME=/home/dockeruser
ENV PATH="/home/dockeruser/.local/bin:${PATH}"

# Force Python stdout and stderr streams to be unbuffered.
ENV PYTHONUNBUFFERED 1

# Add sourcing to your shell startup script
# https://docs.ros.org/en/galactic/Tutorials/Configuring-ROS2-Environment.html#add-sourcing-to-your-shell-startup-script
RUN echo "source /opt/ros/${ROS_DISTRO}/setup.bash" >> ~/.bashrc

# Add `colcon_cd` to your shell startup script
# https://docs.ros.org/en/galactic/Tutorials/Configuring-ROS2-Environment.html#add-colcon-cd-to-your-shell-startup-script
RUN echo "source /usr/share/colcon_cd/function/colcon_cd.sh" >> ~/.bashrc \
    && echo "export _colcon_cd_root=${ros_ws}" >> ~/.bashrc

# This entrypoint is from base image.
# See https://github.com/osrf/docker_images/blob/master/ros/galactic/ubuntu/focal/ros-core/ros_entrypoint.sh
ENTRYPOINT ["/ros_entrypoint.sh"]

CMD ["sleep", "infinity"]
