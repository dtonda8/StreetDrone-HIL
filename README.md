## README
#### Prerequisites
- Ubuntu 22.04 (amd64)
- ROS Humble
- Python 3.10

### Setting up Carla X Autoware
- Note: if you encounter any strange errors, consult [this resolved issue](https://github.com/hatem-darweesh/op_bridge/issues/27)
#### Set up Autoware
1. Create directory called `carlaxautoware` in Home directory:
```bash
mkdir ~/carlaxautoware
cd ~/carlaxautoware
```

2. Install Autoware.universe with all its requirements in  `carlaxautoware`: Follow *How to set up a development environment* and *How to set up a workspace* (up to but excluding `colcon build`) with [Source Installation](https://autowarefoundation.github.io/autoware-documentation/main/installation/autoware/source-installation/#how-to-set-up-a-development-environment)
3. Check out branch release/2023.10, then `colcon build `
```sh
# In Autoware directory ~/carlaxautoware/autoware
git checkout release/2023.10
```

4. Clone OpenPlanner and LIDAR driver in the *universe* folder (/src/universe/external)
```sh
cd ~/carlaxautoware/autoware/src/universe/external
git clone https://github.com/ZATiTech/open_planner.git -b humble
```

```sh
cd ~/carlaxautoware/autoware/src/universe/external
git clone https://github.com/autowarefoundation/awf_velodyne
```

5. Setup `yaml` files 
```sh
cd carlaxautoware/autoware/src/param/autoware_individual_params/individual_params/config/default/
mkdir carla_sensor_kit
cd carla_sensor_kit
wget https://github.com/ZATiTech/open_planner/blob/humble/op_carla_bridge/carla_sensor_kit_launch/carla_sensor_kit_description/config/sensor_kit_calibration.yaml
wget https://github.com/ZATiTech/open_planner/blob/humble/op_carla_bridge/carla_sensor_kit_launch/carla_sensor_kit_description/config/sensors_calibration.yaml
```

6. Build Autoware
```sh
cd ~/carlaxautoware/autoware
colcon build --symlink-install --cmake-args -DCMAKE_BUILD_TYPE=Release
```
- You may need to `colcon build` a few times until there are no more `stderr`s 

#### Set up Carla + additional directories
1. Create sub directory for Carla-related files
```sh
mkdir ~/carlaxautoware/autoware/carla-0.9.15
cd ~/carlaxautoware/autoware/carla-0.9.15
```

2. Download [Carla 0.9.15](https://github.com/carla-simulator/carla/releases) and extract the contents into the directory above
3. Clone these additional repos in `~/carlaxautoware/autoware/carla-0.9.15`:
- `op_agent`
```sh
git clone https://github.com/hatem-darweesh/op_agent.git -b ros2-humble
```

- `scenario_runner` 
```sh
git clone https://github.com/hatem-darweesh/scenario_runner.git
```

- `op_bridge` fork and install its python dependencies
```sh
git clone https://github.com/dtonda8/op_bridge.git -b ros2-humble
cd op_bridge
pip install -r requirements.txt
```
- Note if you encounter errors when install from `requirements.txt`, you'll need to install each python library manually

4. Download relevant Carla`.egg` file
```sh
cd ~/carlaxautoware/carla-0.9.15/PythonAPI/carla/dist
wget https://github.com/dtonda8/op_bridge/blob/ros2-humble/carla-0.9.15-py3.10-linux-x86_64.egg
```

5. Download `pcd` map
```sh
cd ~/carlaxautoware/carla-0.9.15/op_agent/autoware-contents/maps/Town01/
wget https://bitbucket.org/carla-simulator/autoware-contents/src/master/maps/point_cloud_maps/Town01.pcd
mv Town01.pcd pointcloud_map.pcd
```

#### Running Carla X Autoware
First Terminal: 
```
cd ~/carlaxautoware/carla-0.9.15
./CarlaUE4.sh -RenderOffScreen
```

Second Terminal:
```
export OP_BRIDGE_ROOT=~/carlaxautoware/carla-0.9.15/op_bridge
export OP_AGENT_ROOT=~/carlaxautoware/carla-0.9.15/op_agent
export OP_BRIDGE_ROOT=~/carlaxautoware/carla-0.9.15/op_bridge
export AUTOWARE_ROOT=~/carlaxautoware/autoware
export CARLA_ROOT=~/carlaxautoware/carla-0.9.15
export SCENARIO_RUNNER_ROOT=~/carlaxautoware/carla-0.9.15/scenario_runner
export LEADERBOARD_ROOT=~/carlaxautoware/carla-0.9.15/op_bridge/leaderboard 
export TEAM_CODE_ROOT=~/carlaxautoware/carla-0.9.15/op_agent 
export PYTHONPATH=$PYTHONPATH:${CARLA_ROOT}/PythonAPI
export PYTHONPATH=$PYTHONPATH:${CARLA_ROOT}/PythonAPI/util 
export PYTHONPATH=$PYTHONPATH:${CARLA_ROOT}/PythonAPI/carla 
export PYTHONPATH=$PYTHONPATH:${CARLA_ROOT}/PythonAPI/carla/agents
export PYTHONPATH=$PYTHONPATH:${CARLA_ROOT}/PythonAPI/carla/dist/carla-0.9.15-py3.10-linux-x86_64.egg
```

```
cd ~/carlaxautoware/carla-0.9.15/op_bridge/op_scripts
./run_exploration_mode_ros2.sh 
```

##### Broadcasting ROS publishes to LAN (Optional)
Do this before running `/run_exploration_mode_ros2.sh`:
```
export ROS_LOCALHOST_ONLY=2 
export ROS_DOMAIN_ID=2     
```

