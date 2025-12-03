# Shadow Protocol – Multi-Robot Leader–Follower System

Experimental Robotics – Major Exam 2025

## 1. Workspace and Package

- Workspace: `multibot_ws`
- Package: `shadow_protocol`

This package implements the "Shadow Protocol" multi-robot leader–follower system using three TurtleBot3 Waffle Pi robots in Gazebo with ROS 2.

## 2. Prerequisites

- ROS 2 Humble
- TurtleBot3 packages installed and working:
  - `turtlebot3`
  - `turtlebot3_description`
  - `turtlebot3_gazebo`
- TurtleBot3 model:

  ```bash
  export TURTLEBOT3_MODEL=waffle_pi
  ```

## 3. Build Instructions

From the workspace root:

```bash
cd ~/multibot_ws
colcon build --packages-select shadow_protocol
source install/setup.bash
```

## 4. Main Launch Command (Exam Requirement)

Start the entire system (Gazebo + 3 robots + controllers + visualization + RViz) with a single command:

```bash
ros2 launch shadow_protocol start_sim.launch.py
```

This launches:
- Gazebo with `shadow_world.sdf`
- Three TurtleBot3 Waffle Pi robots: leader `tb3_0`, followers `tb3_1` and `tb3_2`
- `robot_state_publisher` for each robot
- Leader keyboard controller
- Two follower controllers
- Visualization node
- RViz with preconfigured view

## 5. Alternative Split Launches (Debugging)

### 5.1. Simulation Only (Gazebo + Followers)

```bash
cd ~/multibot_ws
source install/setup.bash
ros2 launch shadow_protocol gazebo_sim.launch.py
```

This starts Gazebo with three robots and follower controllers only (no keyboard controller).

### 5.2. Manual Leader Teleop

In another terminal:

```bash
cd ~/multibot_ws
source install/setup.bash
ros2 run shadow_protocol keyboard_controller
```

### 5.3. RViz + Visualization

In another terminal (optional):

```bash
cd ~/multibot_ws
source install/setup.bash
ros2 launch shadow_protocol rviz_view.launch.py
```

## 6. Keyboard Controls

Run:

```bash
ros2 run shadow_protocol keyboard_controller
```

Then use the following keys (no Enter):

### 6.1. Leader Motion

- `W` – Move forward
- `S` – Move backward
- `A` – Rotate left
- `D` – Rotate right

The leader stops immediately when no key is pressed.

### 6.2. Dynamic Speed Control

- `Q` – Increase speed
- `E` – Decrease speed

Speed is clamped between **0.05 m/s** and **1.0 m/s**.

### 6.3. Dynamic Safe Distances

- `Z` – Increase `safe_distance_leader` (leader ↔ followers)
- `X` – Decrease `safe_distance_leader`
- `C` – Increase `safe_distance_followers` (between followers)
- `V` – Decrease `safe_distance_followers`

Both distances are clamped between **0.1 m** and **1.0 m**.

### 6.4. Formation Mode

- `T` – Toggle formation mode:
  - `chain` ↔ `triangle`

## 7. Formation Behaviour

### 7.1. Chain Formation (Default)

- Follower1 (`tb3_1`) follows Leader (`tb3_0`) using odometry (`/tb3_0/odom`, `/tb3_1/odom`) and maintains `safe_distance_leader`.
- Follower2 (`tb3_2`) follows Follower1 using `/tb3_1/odom`, `/tb3_2/odom` and maintains `safe_distance_followers`.

Structure:

```text
Leader (tb3_0) --safe_distance_leader--> Follower1 (tb3_1) --safe_distance_followers--> Follower2 (tb3_2)
```

Distance control:
- If leader/preceding robot is far → follower speeds up.
- If too close → follower slows down or reverses.
- If leader stops → follower stops at the defined gap distance.

### 7.2. Triangle Formation

When you press `T` to switch to `triangle` mode:

- Follower1 still follows Leader at `safe_distance_leader`.
- Follower2 switches to follow Leader directly (also at `safe_distance_leader`).

This creates a triangular formation around the leader.

## 8. RViz Visualization

To visualize in RViz, either use the main launch (`start_sim.launch.py`) or the dedicated RViz launch:

```bash
cd ~/multibot_ws
source install/setup.bash
ros2 launch shadow_protocol rviz_view.launch.py
```

In RViz:

- Set `Global Options` → `Fixed Frame` = `odom`.
- Ensure `TF` and `Marker` displays are enabled.
- `Marker` topic: `/shadow_protocol/markers`.

What you see:

- Dynamic safe-distance spheres representing `safe_distance_leader` and `safe_distance_followers`.
- (Depending on TF frame availability) tether-like visualization of the relationship between robots.

> Note: TurtleBot3 Gazebo models use generic TF frame names (`odom`, `base_footprint`) for all three robots, so RViz's `RobotModel` can only render one mesh, even though all three robots exist and move in Gazebo. The multi-robot configuration is clearly visible in Gazebo and through the separate topics (`/tb3_0`, `/tb3_1`, `/tb3_2`).
