# syntax=docker/dockerfile:1
# =============================================================================
# Rolling Tuffy - ROS 2 development image
#
# ROS 2 Jazzy + Gazebo Harmonic + SLAM Toolbox + Nav2 on Ubuntu 24.04.
# Unreal Engine is NOT in here: it runs natively on Windows and talks to ROS in
# this container over a WebSocket (ws://127.0.0.1:<port>).
#
# The repo is NOT copied into the image. compose.yaml mounts it at
#   ~/rolling_tuffy_ws/src/rolling_tuffy
# so edits on Windows / WSL show up in the container immediately.
#
# Build:  cd docker && docker compose build
# =============================================================================

# ---- Base image -------------------------------------------------------------
# Official Open Robotics image: Ubuntu 24.04 with ROS 2 Jazzy desktop-full
# (core ROS, RViz, demos, visualization tools) already installed.
FROM osrf/ros:jazzy-desktop-full

# ---- Build settings ---------------------------------------------------------
# Name and IDs of the non-root user created below. UID/GID 1000 matches the
# first user in WSL, so files written into the repo belong to you, not root.
# compose.yaml can override these.
ARG USERNAME=dev
ARG USER_UID=1000
ARG USER_GID=1000
# Stop apt from asking questions during the build (build-time only).
ARG DEBIAN_FRONTEND=noninteractive

# ---- System + ROS packages --------------------------------------------------
# One RUN so the apt cache cleanup at the end actually shrinks the image.
RUN apt-get update && apt-get install -y --no-install-recommends \
        # Simulation: Gazebo Harmonic + the ROS <-> Gazebo bridge
        ros-jazzy-ros-gz \
        # Mapping and navigation
        ros-jazzy-slam-toolbox \
        ros-jazzy-navigation2 \
        ros-jazzy-nav2-bringup \
        # Robot model + manual testing tools
        ros-jazzy-xacro \
        ros-jazzy-joint-state-publisher-gui \
        ros-jazzy-teleop-twist-keyboard \
        liburdfdom-tools \
        # Build tools
        python3-colcon-common-extensions \
        python3-rosdep \
        # Python libraries our nodes use (math, YAML config, Unreal bridge, tests)
        python3-numpy \
        python3-yaml \
        python3-websockets \
        python3-pytest \
        # Debugging: ports/processes, GPU check, shell script linter
        iproute2 procps psmisc \
        mesa-utils \
        shellcheck \
        # Everyday tools: editors (nano for everyone, neovim for vim users),
        # tmux for several terminals in one shell, git, sudo, Tab completion, pager
        nano neovim tmux git sudo bash-completion less \
    && rm -rf /var/lib/apt/lists/*

# ---- Non-root user ----------------------------------------------------------
# Ubuntu 24.04 images already ship a user "ubuntu" with UID 1000: rename it to
# $USERNAME. Otherwise create the user from scratch. Then allow passwordless
# sudo so you can still install extra packages inside the container.
RUN if id -u ubuntu >/dev/null 2>&1; then \
        usermod -l ${USERNAME} -d /home/${USERNAME} -m ubuntu \
        && groupmod -n ${USERNAME} ubuntu; \
    else \
        groupadd --gid ${USER_GID} ${USERNAME} \
        && useradd -m -s /bin/bash --uid ${USER_UID} --gid ${USER_GID} ${USERNAME}; \
    fi \
    && echo "${USERNAME} ALL=(root) NOPASSWD:ALL" > /etc/sudoers.d/${USERNAME} \
    && chmod 0440 /etc/sudoers.d/${USERNAME}

# Everything below runs as the normal user.
USER ${USERNAME}

# ---- Workspace + shell setup ------------------------------------------------
# 1. Create the colcon workspace (owned by $USERNAME; compose stores it in a volume).
# 2. Download rosdep's package database (saved per user in ~/.ros).
# 3. Add lines to ~/.bashrc so every shell has ROS (and our built packages)
#    loaded, uses ROS domain 43, and starts in the repo folder.
RUN mkdir -p /home/${USERNAME}/rolling_tuffy_ws/src \
    && rosdep update --rosdistro jazzy \
    && printf '%s\n' \
        '# >>> rolling_tuffy >>>' \
        'source /opt/ros/jazzy/setup.bash' \
        '[ -f "$HOME/rolling_tuffy_ws/install/setup.bash" ] && source "$HOME/rolling_tuffy_ws/install/setup.bash"' \
        'export ROS_DOMAIN_ID=${ROS_DOMAIN_ID:-43}' \
        'cd "$HOME/rolling_tuffy_ws/src/rolling_tuffy" 2>/dev/null || true' \
        '# <<< rolling_tuffy <<<' >> /home/${USERNAME}/.bashrc

# ---- Container defaults -----------------------------------------------------
# Start in the repo, and just stay running so we can open shells with
# `docker compose exec ros bash` and run ROS by hand.
WORKDIR /home/${USERNAME}/rolling_tuffy_ws/src/rolling_tuffy
CMD ["sleep", "infinity"]
