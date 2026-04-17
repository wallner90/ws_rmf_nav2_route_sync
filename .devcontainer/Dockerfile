FROM osrf/ros:jazzy-desktop-full

ENV DEBIAN_FRONTEND=noninteractive
ARG ZENOH_VERSION=1.5.0

RUN apt-get update \
    && apt-get -y install --no-install-recommends \
    ros-${ROS_DISTRO}-nav2-bringup \
    ros-${ROS_DISTRO}-ros-gz \
    ros-${ROS_DISTRO}-rmw-cyclonedds-cpp \
    ca-certificates \
    python3-pip \
    unzip \
    wget \
    # Clean up
    && apt-get autoremove -y \
    && apt-get clean -y \
    && rm -rf /var/lib/apt/lists/*


RUN wget -O /tmp/zenoh-plugin-ros2dds.zip \
      https://github.com/eclipse-zenoh/zenoh-plugin-ros2dds/releases/download/${ZENOH_VERSION}/zenoh-plugin-ros2dds-${ZENOH_VERSION}-x86_64-unknown-linux-gnu-standalone.zip \
    && unzip -j /tmp/zenoh-plugin-ros2dds.zip 'zenoh-bridge-ros2dds' -d /usr/local/bin \
    && chmod +x /usr/local/bin/zenoh-bridge-ros2dds \
    && rm -f /tmp/zenoh-plugin-ros2dds.zip

RUN wget -O /tmp/zenoh.zip \
      https://github.com/eclipse-zenoh/zenoh/releases/download/${ZENOH_VERSION}/zenoh-${ZENOH_VERSION}-x86_64-unknown-linux-gnu-standalone.zip \
    && unzip -j /tmp/zenoh.zip 'zenohd' -d /usr/local/bin \
    && chmod +x /usr/local/bin/zenohd \
    && rm -f /tmp/zenoh.zip

RUN python3 -m pip install --break-system-packages --no-cache-dir \
    nudged \
    eclipse-zenoh==1.5.0 \
    pycdr2 \
    rosbags

# Set up auto-source of workspace for ros 
ARG USERNAME=ros
ARG USER_UID=1000
ARG USER_GID=$USER_UID

# Find and replace any user with matching UID
RUN set -eux; \
    existing_user=$(getent passwd "$USER_UID" | cut -d: -f1 || true); \
    if [ -n "$existing_user" ]; then \
        usermod -l $USERNAME -d /home/$USERNAME -m $existing_user && \
        groupmod -n $USERNAME $existing_user && \
        echo "Renamed $existing_user to $USERNAME"; \
    else \
        groupadd --gid $USER_GID $USERNAME && \
        useradd -s /bin/bash --uid $USER_UID --gid $USER_GID -m $USERNAME && \
        echo "Created new user $USERNAME"; \
    fi
# Ensure necessary directories and permissions
RUN mkdir -p /home/$USERNAME /run/user/$USER_UID && \
    chown -R $USER_UID:$USER_GID /home/$USERNAME /run/user/$USER_UID

# Add sudo support for the non-root user
RUN echo "$USERNAME ALL=(ALL) NOPASSWD:ALL" > /etc/sudoers.d/$USERNAME \
    && chmod 0440 /etc/sudoers.d/$USERNAME \
    && touch /home/$USERNAME/.sudo_as_admin_successful

ARG WORKSPACE
RUN echo "if [ -f /opt/ros/${ROS_DISTRO}/setup.bash ]; then source /opt/ros/${ROS_DISTRO}/setup.bash; fi" >> /home/$USERNAME/.bashrc \
    && echo "if [ -f ${WORKSPACE}/install/setup.bash ]; then source ${WORKSPACE}/install/setup.bash; fi" >> /home/$USERNAME/.bashrc
