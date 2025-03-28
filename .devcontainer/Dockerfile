FROM ct2034/vnc-ros-kinetic-full

# some ros key stuff
RUN sudo sh -c 'echo "deb http://packages.ros.org/ros/ubuntu $(lsb_release -sc) main" > /etc/apt/sources.list.d/ros-latest.list'
RUN curl -s https://raw.githubusercontent.com/ros/rosdistro/master/ros.asc | sudo apt-key add -

# some maintenance
RUN sudo rm -f /etc/apt/sources.list.d/realsense-public.list
RUN sudo sed -i '/opensuse/ s/^/#/' /etc/apt/sources.list.d/*.list

RUN sudo apt-get update && sudo apt-get install -y apt-transport-https ca-certificates curl software-properties-common

RUN sudo mkdir -p /etc/apt/keyrings
RUN curl -sSf https://librealsense.intel.com/Debian/librealsense.pgp | sudo tee /etc/apt/keyrings/librealsense.pgp > /dev/null
RUN echo "deb [signed-by=/etc/apt/keyrings/librealsense.pgp] https://librealsense.intel.com/Debian/apt-repo `lsb_release -cs` main" | sudo tee /etc/apt/sources.list.d/librealsense.list

RUN sudo apt-get update && sudo apt-get dist-upgrade -y --allow-unauthenticated

### Core Linux tools
RUN sudo apt-get install -y \
  apt-utils lsb-release sudo unzip wget ssh vim curl tmux software-properties-common gnupg2 apt-transport-https

### ROS tools
RUN sudo apt-get install -y librealsense2-dkms python-dev python-dev git python-pip python-catkin-tools python-rosdep python-rosinstall python-rosinstall-generator python-wstool build-essential

### User and permissions
ARG user=ir  # Intelligent Robots!
ARG group=ir
ARG uid=1000
ARG gid=1000
ARG home=/home/${user}
RUN mkdir -p /etc/sudoers.d \
    && groupadd -g ${gid} ${group} \
    && useradd -d ${home} -u ${uid} -g ${gid} -m -s /bin/bash ${user} \
    && echo "${user} ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/sudoers_${user}  # passwordless sudo
USER ${user}
RUN sudo usermod -a -G video ${user}
WORKDIR ${home}
ENV HOME ${home}
ENV USER ${user}

### Setup ROS environment
RUN rosdep update
RUN printf '\nsource /opt/ros/kinetic/setup.bash' >> ${home}/.bashrc

### Lab 1 dependency
RUN sudo apt-get install -y --no-install-recommends ros-kinetic-turtlebot-stage ros-kinetic-joint-state-publisher-gui ros-kinetic-turtlesim

### Docker image entrypoint
RUN sudo sed -i -e\
    's/chown -R $USER:$USER ${HOME}/chown -R $USER:$USER {.config,.gtkrc-2.0,.asoundrc}/g'\
    /startup.sh
COPY entrypoint.bash /usr/local/bin/entrypoint
COPY start-novnc.bash /usr/local/bin/start-novnc
ENTRYPOINT ["entrypoint"]
CMD ["start-novnc"]
