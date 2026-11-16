# Yuhesen FR-max platform configuration for Autoware

Vehicle geometry, visualization, sensor descriptions, and launch composition for adapting the Yuhesen FR-max Ackermann robot to the [CARLA–Autoware simulation project](https://github.com/gokcerSnmzck/autoware_carla_leaderboard_custom).

This repository describes the platform inside Autoware. The separate [carla_fr_maxv10 repository](https://github.com/gokcerSnmzck/carla_fr_maxv10) provides the Unreal vehicle assets inside CARLA; the main project provides agents, command conversion and scenario definitions.

## Engineering contribution

The platform adaptation expresses the compact robot's footprint, steering envelope, visualization and sensor transforms using ROS 2 packages and Autoware launch conventions. It configures existing Autoware components rather than implementing a new localization or perception algorithm. The import commit is by Gökcer Sönmezocak; sensor package manifests and launch files retain TUM/Gemb Kaljavesi and Tier IV attribution. Original authorship of every imported component is not claimed.

## Package structure

The outer directories group packages; they are not themselves ROS 2 packages. Recursive colcon discovery should find all five:

| Package | Location and purpose |
|---|---|
| `yuhesen_fr_max_vehicle_launch` | `yuhesen_fr_max_vehicle_launch/yuhesen_fr_max_vehicle_launch`: vehicle interface launch; currently an empty interface placeholder. |
| `yuhesen_fr_max_vehicle_description` | `yuhesen_fr_max_vehicle_launch/yuhesen_fr_max_vehicle_description`: YAML geometry, simulator parameters, Xacro, LFS mesh. |
| `yuhesen_fr_max_sensor_kit_launch` | `yuhesen_fr_max_sensor_kit_launch/yuhesen_fr_max_sensor_kit_launch`: sensing entry point, LiDAR composition, IMU/GNSS paths and camera relay configuration. |
| `yuhesen_fr_max_sensor_kit_description` | `yuhesen_fr_max_sensor_kit_launch/yuhesen_fr_max_sensor_kit_description`: sensor Xacro and extrinsic YAML. |
| `yuhesen_fr_max_common_sensor_launch` | `yuhesen_fr_max_sensor_kit_launch/yuhesen_fr_max_common_sensor_launch`: vehicle/mirror cloud crop-filter composition. |

## Vehicle and sensor parameters

[Vehicle geometry](yuhesen_fr_max_vehicle_launch/yuhesen_fr_max_vehicle_description/config/vehicle_info.param.yaml) sets wheelbase **0.85 m**, tread **0.645 m**, wheel radius **0.21 m**, wheel width **0.12 m**, front/rear overhang **0.375 m**, lateral overhang **0.0875 m**, height **0.52 m**, and maximum tire angle **0.436 rad** (approximately 25°). Derived length and width are **1.60 m × 0.82 m**. These are configured values, not independent physical measurements.

[Simulator parameters](yuhesen_fr_max_vehicle_launch/yuhesen_fr_max_vehicle_description/config/simulator_model.param.yaml) select `DELAY_STEER_ACC_GEARED`, a 25 ms sampling interval and steering/acceleration delays and time constants. They configure Autoware's simulator model; they do not set CARLA Unreal physics. The configured velocity limit of 50 m/s is not an FR-max rated speed or validated operating envelope.

[Extrinsics](yuhesen_fr_max_sensor_kit_launch/yuhesen_fr_max_sensor_kit_description/config/sensor_kit_calibration.yaml) place front/rear LiDAR at x=±0.75 m, z=0.55 m relative to `sensor_kit_base_link`, GNSS at z=0.52 m, IMU at (0.1, 0, 0.3), and front camera at (0.7, 0, 0.52). [The kit transform](yuhesen_fr_max_sensor_kit_launch/yuhesen_fr_max_sensor_kit_description/config/sensors_calibration.yaml) adds x=0.425 m from rear-axle `base_link`. Account for this composition when matching the CARLA actor origin; equal local offsets alone do not establish calibration agreement.

[Sensor Xacro](yuhesen_fr_max_sensor_kit_launch/yuhesen_fr_max_sensor_kit_description/urdf/sensor_kit.xacro) uses VLS-128 macros for both LiDAR frames and an 800×600 camera. The main agent requests 32/16 channels and an 800×400 camera. The GNSS visualization uses an IMU macro. These descriptions require alignment with the chosen simulated sensor contract before claiming runtime equivalence.

LiDAR launch configuration takes `/sensor/lidar/front` and `/sensor/lidar/rear`, crops vehicle and mirror bounds using external `autoware_pointcloud_preprocessor` components, and configures concatenation/time synchronization into `base_link` with velocity feedback. The sensing entry point leaves the camera relay include commented out; in the main E2E workflow the C++ bridge publishes camera topics directly.

## Installation

Use a complete Autoware Universe workspace with ROS 2 Humble and its selected release's dependencies. The earlier README stated compatibility with 1.7.1; no pinned Autoware checkout or build record in this repository verifies that guarantee. Package and plugin names must be checked against the actual workspace.

```bash
export AUTOWARE_WS=/path/to/autoware
git lfs install
mkdir -p "$AUTOWARE_WS/src/vehicle"
cd "$AUTOWARE_WS/src/vehicle"
git clone https://github.com/gokcerSnmzck/carla_yuhesen_fr_max.git
git -C carla_yuhesen_fr_max lfs pull
git -C carla_yuhesen_fr_max lfs fsck
cd "$AUTOWARE_WS"
source /opt/ros/humble/setup.bash
colcon list | rg yuhesen_fr_max
rosdep install --from-paths src --ignore-src -r -y --rosdistro humble
colcon build --symlink-install --packages-up-to \
  yuhesen_fr_max_vehicle_launch yuhesen_fr_max_vehicle_description \
  yuhesen_fr_max_sensor_kit_launch yuhesen_fr_max_sensor_kit_description \
  yuhesen_fr_max_common_sensor_launch
source install/setup.bash
```

All five packages are needed; building just the launch packages does not install their description assets. The COLLADA mesh is one Git LFS object (317,952,716 bytes). The payload was downloaded and SHA-256/size verified during portfolio review; COLLADA metadata identifies version 1.4.1, centimetre units and Z-up. The repository checkout may still contain a pointer until `git lfs pull` completes.

## Integration and verification

After preparing the full Autoware stack, checking launch expansion and installing maps:

```bash
export MAP_PATH=/path/to/Town01
ros2 launch autoware_launch e2e_simulator.launch.xml --show-args
ros2 launch autoware_launch e2e_simulator.launch.xml \
  vehicle_model:=yuhesen_fr_max_vehicle \
  sensor_model:=yuhesen_fr_max_sensor_kit map_path:="$MAP_PATH"
```

Launch composition still requires verification: concatenation uses inconsistent container argument names, `use_concat_filter` depends on enclosing scope, the LiDAR container declares `container_name` twice, and some manifests use old unprefixed package names while launch files use `autoware_*`. Hardware driver/GNSS conversion branches coexist with simulation feedback; verify effective publishers and driver flags. No sensor calibration or launch behavior was changed during the documentation update.

See the main project's [setup prerequisites](https://github.com/gokcerSnmzck/autoware_carla_leaderboard_custom/blob/main/docs/SETUP.md#execution-prerequisites), [architecture](https://github.com/gokcerSnmzck/autoware_carla_leaderboard_custom/blob/main/docs/ARCHITECTURE.md), and [scenario definitions](https://github.com/gokcerSnmzck/autoware_carla_leaderboard_custom/blob/main/docs/SIMULATION_SCENARIOS.md). These links target the portfolio documents once published. Static syntax/metadata checks do not establish a successful colcon build, expanded Xacro, TF alignment or simulation run.

## Author and attribution

Platform adaptation and integration: **Gökcer Sönmezocak**, M.Eng. Embedded Systems Engineering, FH Dortmund — University of Applied Sciences and Arts.

Preserve [LGPLv3 licensing](LICENSE.md), package maintainer records, and TUM/Tier IV source notices, including Apache-2.0 headers where present. Mesh metadata does not establish original CAD authorship. These research configurations do not establish hardware safety or real-robot readiness.
