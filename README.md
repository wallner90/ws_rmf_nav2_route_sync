# Open-RMF ↔ Nav2 Route Sync Demo

Demo workspace for the Open-RMF ↔ Nav2 route synchronization example on ROS 2 Jazzy.

## Overview

![Open-RMF ↔ Nav2 route sync architecture overview](media/rmf_nav2_route_sync_architecture.png)

Open-RMF and Nav2 use the same route graph. The graph is generated at build time
from an RMF site description and loaded into Nav2 on startup. At runtime, any lane
closures or reopenings issued through Open-RMF are propagated to Nav2, so both
systems always agree on which routes are available.

Nav2 is configured with the SMAC planner and the Regulated Pure Pursuit (RPP)
controller so the robot tracks the route graph closely.

## Setup

This workspace is intended to be used in the provided devcontainer. New terminals
automatically source ROS 2 and the workspace overlay.

Before running the demo, install `zenohd` on the host system by following the
[official guide](https://zenoh.io/docs/getting-started/installation/#ubuntu-or-any-debian).
The host terminal used below must be able to launch `zenohd` directly.

Open the devcontainer and run:

```bash
./setup.sh
```

This imports the repositories listed in `src/ros2.repos` and installs the required dependencies.

Then build the workspace:

```bash
./build.sh
```

Re-run `./build.sh` after any source code changes.

## Running the Demo

After building the workspace, use the following terminals.

### Host terminal

Start the zenoh router on the host machine:

```bash
zenohd
```

### Terminal 1: Simulation

```bash
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
ros2 launch rmf_nav2_route_demo nav2.launch.py
```

### Terminal 2: zenoh bridge

```bash
export RMW_IMPLEMENTATION=rmw_cyclonedds_cpp
zenoh-bridge-ros2dds -c $(ros2 pkg prefix free_fleet_examples)/share/free_fleet_examples/config/zenoh/nav2_tb3_zenoh_bridge_ros2dds_client_config.json5
```

### Terminal 3: RMF + fleet adapter

```bash
export ROS_DOMAIN_ID=55
ros2 launch rmf_nav2_route_demo rmf.launch.xml
```

## Demo Actions

This demo illustrates that lane closures are synchronized between RMF and the Nav2
route graph. Follow the steps below in order to see the difference.

The same closure state is also propagated to the Nav2 route server, so graph-based
navigation goals sent directly through Nav2 will avoid the closed lanes until they
are reopened.

### 1. Dispatch with lanes open

With all lanes open (the default), dispatch a task. The robot will traverse the
corridor through lanes `20` and `21` because it is part of the shortest path.

```bash
export ROS_DOMAIN_ID=55
ros2 run rmf_demos_tasks dispatch_go_to_place -p north_east
```

### 2. Close lanes

```bash
export ROS_DOMAIN_ID=55
ros2 topic pub --once /lane_closure_requests rmf_fleet_msgs/msg/LaneRequest \
	"{fleet_name: 'turtlebot3', close_lanes: [20, 21], open_lanes: []}"
```

### 3. Return to start with lanes closed

Send the robot back to its starting position. Because lanes `20` and `21` are now
closed, the robot cannot retrace the path it took to reach `north_east` and will use
an alternate route instead.

```bash
export ROS_DOMAIN_ID=55
ros2 run rmf_demos_tasks dispatch_go_to_place -p tb3_charger
```

### 4. Reopen lanes

```bash
export ROS_DOMAIN_ID=55
ros2 topic pub --once /lane_closure_requests rmf_fleet_msgs/msg/LaneRequest \
	"{fleet_name: 'turtlebot3', close_lanes: [], open_lanes: [20, 21]}"
```

After reopening the lanes, subsequent RMF dispatches and Nav2 graph-based goals can
use that route again.

## License

This project is licensed under the Apache License 2.0. See [LICENSE](LICENSE) for details.