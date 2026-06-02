# CARLA Yuhesen FR-max Autoware Wrapper

This repository contains the Autoware vehicle and sensor kit descriptions, along with launch configurations, for the **Yuhesen FR-max** robot model. It is fully compatible with **Autoware Universe 1.7.1**.

## Repository Structure
* \`yuhesen_fr_max_vehicle_launch\` - Vehicle parameters, kinematics, and vehicle-level configurations.
* \`yuhesen_fr_max_sensor_kit_launch\` - Extrinsic sensor calibrations, sensor macro layouts, and launch configurations.

## Installation & Compilation

Navigate to the source directory of your Autoware Universe workspace and clone this repository:
\`\`\`bash
cd /path/to/autoware/src
git clone https://github.com/gokcerSnmzck/carla_yuhesen_fr_max.git
\`\`\`

Build the specific packages using colcon:
\`\`\`bash
cd ..
colcon build --symlink-install --packages-select yuhesen_fr_max_vehicle_launch yuhesen_fr_max_sensor_kit_launch
\`\`\`
