# Docker Container for ROS 2 Galactic with Gazebo, SLAM Toolbox, and Navigation2

## What is this?

*   A Docker container environment based on `osrf/ros:galactic-desktop`.
*   Includes ROS 2 Galactic, Gazebo simulator, `slam_toolbox`, and `navigation2` packages.
*   Sets up a non-root user (`dockeruser`) for better security and file permissions.
*   Configured for GUI application forwarding using WSLg (on Windows/WSL2) or standard X11 forwarding (on Linux/macOS).
*   Mounts local directories for project files (`./`), a development workspace (`./dev_ws`), ROS logs (`./docker/ros2/.ros`), and Gazebo configurations (`./docker/ros2/.gazebo`) for persistence and ease of development.

## Prerequisites

*   Docker ([Install Docker Engine](https://docs.docker.com/engine/install/))
*   Docker Compose ([Install Docker Compose](https://docs.docker.com/compose/install/))
*   **For GUI Applications (Gazebo, Rviz, Turtlesim, etc.):**
    *   **Windows:** WSL2 with WSLg enabled (usually default on recent Windows 10/11).
    *   **Linux:** An X server running on the host and `xhost` configured to allow connections (e.g., `xhost +local:docker`).
    *   **macOS:** An X server like XQuartz installed and running, with appropriate security settings configured.

## Getting Started

1.  **Clone the repository:**
    ```bash
    git clone https://github.com/woreom/ros2-gazebo-docker.git
    cd ros2-gazebo-docker
    ```

2.  **Prepare Local Directories (Optional but Recommended):**
    Create the following directories in the project root if they don't exist. This prevents Docker from creating them as root.
    ```bash
    mkdir -p dev_ws docker/ros2/.ros docker/ros2/.gazebo
    ```

3.  **Build the Docker image:**
    *   The Docker image creates a user `dockeruser` inside the container. To avoid permission issues with mounted volumes, build the image using your host's user and group ID.
    ```bash
    # On Linux/macOS/WSL:
    docker compose build --build-arg USER_ID=$(id -u) --build-arg GROUP_ID=$(id -g)

    # On Windows (Git Bash/CMD/PowerShell without WSL):
    # You might need to find your UID/GID manually or use the default (1000).
    # If unsure or using default, simply run:
    # docker compose build
    ```
    *   The default ROS workspace inside the container is set to `/home/dockeruser/dev_ws`.

4.  **Run the container:**
    ```bash
    docker compose up -d
    ```
    This starts the `ros2` service in detached mode. The necessary environment variables (`DISPLAY`, etc.) and volume mounts for GUI forwarding (especially for WSLg) are handled by `docker-compose.yml`.

5.  **Access the container shell:**
    ```bash
    docker compose exec ros2 bash
    ```
    You are now inside the container's bash shell as the `dockeruser`. The ROS 2 environment (`/opt/ros/galactic/setup.bash`) is automatically sourced on login via `~/.bashrc`.

## Container Configuration Details

*   **Volumes:**
    *   `./:/home/dockeruser/project`: Your project directory is mounted read-write.
    *   `./dev_ws:/home/dockeruser/dev_ws`: This is the intended location for your custom ROS 2 packages. Place your source code in `./dev_ws/src` on your host machine.
    *   `./docker/ros2/.ros:/home/dockeruser/.ros`: Persists ROS logs and cache.
    *   `./docker/ros2/.gazebo/:/home/dockeruser/.gazebo`: Persists Gazebo models, logs, and cache.
    *   `/mnt/wslg` & `/tmp/.X11-unix`: Used for GUI forwarding (WSLg and X11 respectively).
*   **User:** Operates as `dockeruser` (UID/GID match host if built with args).
*   **Networking:** Uses host networking by default (check `docker-compose.yml` if different).
*   **`.bashrc`:** Automatically sources ROS 2 Galactic setup and configures `colcon_cd` for the `/home/dockeruser/dev_ws` workspace.

## Usage Examples within the Container

*(Remember to run `docker compose exec ros2 bash` first)*

### Basic ROS 2 (Turtlesim)

1.  **Start Turtlesim:** (Requires GUI Forwarding)
    ```bash
    ros2 run turtlesim turtlesim_node
    ```
2.  **Control the turtle (in a *separate* terminal):**
    ```bash
    docker compose exec ros2 bash
    # Inside the new shell:
    ros2 run turtlesim turtle_teleop_key
    ```

### Gazebo Simulation

1.  **Launch Gazebo with a demo world:** (Requires GUI Forwarding)
    ```bash
    # Launch Gazebo via ROS 2 to include ROS plugins and handle simulation time
    ros2 launch gazebo_ros gazebo.launch.py world:=/opt/ros/galactic/share/gazebo_plugins/worlds/gazebo_ros_diff_drive_demo.world
    ```
    *(You can omit the `world:=...` argument to launch an empty world)*

2.  **Control a robot (e.g., the diff drive demo):** (In a separate terminal)
    ```bash
    docker compose exec ros2 bash
    # Inside the new shell:
    ros2 topic pub /demo/cmd_demo geometry_msgs/Twist '{linear: {x: 0.5}, angular: {z: 0.5}}' --once
    ```

### SLAM Toolbox and Navigation2

*Note: These require specific robot configurations (URDF, sensors) and launch files.*

1.  **Example: Launching Navigation2 with SLAM (using TurtleBot3 simulation defaults):**
    *(This specific launch file might require additional TurtleBot3 packages installed in your `./dev_ws` workspace - adjust as needed for your robot setup)*

    *   First, ensure Gazebo is running with your robot model and sensors (e.g., using a custom launch file).
    *   Then, in a new terminal:
        ```bash
        docker compose exec ros2 bash
        # Inside the new shell:
        # If you have custom packages in dev_ws, source them:
        # source /home/dockeruser/dev_ws/install/setup.bash
        # Example launch command (may need customization):
        ros2 launch nav2_bringup tb3_simulation_launch.py slam:=True use_sim_time:=True
        ```

## Custom Workspace Development (`dev_ws`)

1.  Place your custom ROS 2 package source code in the `./dev_ws/src` directory on your host machine.
2.  Access the container: `docker compose exec ros2 bash`
3.  Navigate to the workspace: `cd ~/dev_ws`
4.  Build your packages: `colcon build --symlink-install` (or other colcon options)
5.  Source the workspace overlay (needed in *every new terminal* where you want to use your custom packages):
    ```bash
    source ~/dev_ws/install/setup.bash
    ```
    *(You can add this line to `~/.bashrc` inside the container if you always want it sourced)*
6.  Run your custom nodes: `ros2 run your_package_name your_node_name`

## Managing the Container

*   **Stop the container:**
    ```bash
    docker compose stop
    ```
*   **Stop and remove the container (and network if created):**
    ```bash
    docker compose down
    ```
*   **View logs:**
    ```bash
    docker compose logs -f ros2
    ```
*   **Access the container as root (use with caution):**
    ```bash
    docker compose exec --user root ros2 bash
    ```