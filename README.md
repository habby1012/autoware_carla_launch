# autoware_carla_launch

The package includes launch file to run Autoware, Carla agent, and bridge ([zenoh-bridge-dds](https://github.com/eclipse-zenoh/zenoh-plugin-dds) + [zenoh_carla_bridge](https://github.com/evshary/zenoh_carla_bridge)).

## Demo

[![IMAGE ALT TEXT](http://img.youtube.com/vi/UFBRMqJ2r0w/0.jpg)](https://youtu.be/UFBRMqJ2r0w "Run multiple vehicles with Autoware in Carla")

# Architecture

![image](https://user-images.githubusercontent.com/456210/232400804-e0e0a755-0f6d-4873-a8ad-f1188011c993.png)

# Prerequisites

Make sure you meet the following system requirements

* Ubuntu 20.04
* Carla 0.9.13

Install rocker for containers

```shell
sudo apt install docker.io python3-rocker
```

# Build

## Build the container for Carla bridge

* Enter into docker

```shell
./run-bridge-docker.sh
```

* Build Carla bridge

```shell
cd autoware_carla_launch
source env.sh
make prepare_bridge
make build_bridge
```

## Build the container for Zenoh+Autoware

* Enter into docker

```shell
./run-autoware-docker.sh
```

* Build Carla bridge

```shell
cd autoware_carla_launch
source env.sh
make prepare_autoware
make build_autoware
```

# Usage

## Carla with Autoware

The section shows how to run Autoware in Carla simulator.

1. Run Carla simulator (In native host)

```shell
./CarlaUE4.sh -quality-level=Epic -world-port=2000 -RenderOffScreen
```

2. Run Carla Bridge and Python Agent (In Carla bridge container)

```shell
ros2 launch autoware_carla_launch carla_bridge.launch.xml
```

3. Run zenoh-bridge-dds and Autoware (In Autoware container)

```shell
ros2 launch autoware_carla_launch autoware_zenoh.launch.xml
```

## Run multiple vehicles with Autoware in Carla at the same time

* Since running two Autoware will cause port conflict, we need to do some modification.
  - Modify `src/universe/autoware.universe/launch/tier4_planning_launch/launch/scenario_planning/lane_driving/behavior_planning/behavior_planning.launch.py` (About line 177) 

```diff
+ import random
....
            {
                "bt_tree_config_path": [
                    FindPackageShare("behavior_path_planner"),
                    "/config/behavior_path_planner_tree.xml",
                ],
+               "groot_zmq_publisher_port": random.randint(2000, 4000),
+               "groot_zmq_server_port": random.randint(2000, 4000),
                "planning_hz": 10.0,
            },
```

* Able to spawn second vehicle into Carla.
  - Modify `src/autoware_carla_launch/launch/carla_bridge.launch.xml` (About line 7)

```diff
-<executable cmd="poetry run python3 main.py --host $(env CARLA_SIMULATOR_IP) --sync --rolename $(env VEHICLE_NAME)" cwd="$(env AUTOWARE_CARLA_ROOT)/external/zenoh_carla_bridge/carla_agent" output="screen" />
+<executable cmd="poetry run python3 main.py --host $(env CARLA_SIMULATOR_IP) --sync --rolename $(env VEHICLE_NAME) --position 87.687683,145.671295,0.300000,0.000000,90.000053,0.000000" cwd="$(env AUTOWARE_CARLA_ROOT)/external/zenoh_carla_bridge/carla_agent" output="screen" />
+<executable cmd="poetry run python3 main.py --host $(env CARLA_SIMULATOR_IP) --rolename 'v2' --position 92.109985,227.220001,0.300000,0.000000,-90.000298,0.000000" cwd="$(env AUTOWARE_CARLA_ROOT)/external/zenoh_carla_bridge/carla_agent" output="screen" />
```

* Spawn vehicles into Carla
  - Run Carla
  - Run `ros2 launch autoware_carla_launch carla_bridge.launch.xml`

* Run Autoware twice:
  - 1st: `ROS_DOMAIN_ID=1 VEHICLE_NAME="v1" ros2 launch autoware_carla_launch autoware_zenoh.launch.xml`
  - 2nd: `ROS_DOMAIN_ID=2 VEHICLE_NAME="v2" ros2 launch autoware_carla_launch autoware_zenoh.launch.xml`

* Now there are two rviz with separated Autoware at the same time. You can control them separately!

# FAQ

* [How to change the map](carla_map/README.md)

# Known Issues

* Generate the map: Now we're using the maps generated by Hatem.

