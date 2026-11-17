# Rolling Tuffy

CPSC 490/491 capstone. For now this repo only holds the shared ROS 2 dev environment.

- `docker/` - ROS 2 Jazzy + Gazebo Harmonic + SLAM Toolbox + Nav2 container (runs in WSL2).
- Unreal Engine 5.8 runs natively on Windows.

Inside the container the repo lives at `~/rolling_tuffy_ws/src/rolling_tuffy`
(standard ROS workspace layout), and you build from `~/rolling_tuffy_ws`.

Day to day (Ubuntu terminal):

```bash
cd docker
docker compose up -d              # start
docker compose exec ros bash      # step inside
cd ~/rolling_tuffy_ws && colcon build --symlink-install   # build our packages
docker compose down               # stop
```
