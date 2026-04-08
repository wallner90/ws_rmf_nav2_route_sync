# RMF <-> Nav2 Route Sync Demo

Demo workspace for the RMF <-> Nav2 route synchronization example on ROS 2 Jazzy.

In this demo, Nav2 is configured with the SMAC planner and the Regulated Pure Pursuit
(RPP) controller so the robot tracks the route graph closely.

This workspace is intended to be used in the provided devcontainer. New terminals
automatically source ROS 2 and the workspace overlay.

## Setup

Open the devcontainer and run:

```bash
./setup.sh
```

This imports the repositories listed in `src/ros2.repos` and installs the required dependencies.

If you modify source code and need to rebuild the workspace:

```bash
./build.sh
```

## Running the Demo

After `./setup.sh`, use the following terminals.

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
route graph. When the example dispatch is issued while lanes `20` and `21` are open,
the robot will traverse that corridor because it is part of the shortest available
path. If those lanes are closed before dispatch, RMF replans the task and the robot
uses an alternate route.

Because the task is issued through RMF, this behavior confirms that RMF is aware of
the closure state. The same closure information is also propagated to the Nav2 route
server, so graph-based navigation goals sent directly through Nav2 will continue to
avoid the closed lanes until they are reopened.

### Close lanes

```bash
export ROS_DOMAIN_ID=55
ros2 topic pub --once /lane_closure_requests rmf_fleet_msgs/msg/LaneRequest \
	"{fleet_name: 'turtlebot3', close_lanes: [20, 21], open_lanes: []}"
```

### Dispatch example task

```bash
export ROS_DOMAIN_ID=55
ros2 run rmf_demos_tasks dispatch_go_to_place -p north_east
```

Expected behavior:

- If lanes `20` and `21` are open, the dispatched task should use that route.
- If lanes `20` and `21` are closed before dispatching, the robot should reroute around them.

### Open lanes

```bash
export ROS_DOMAIN_ID=55
ros2 topic pub --once /lane_closure_requests rmf_fleet_msgs/msg/LaneRequest \
	"{fleet_name: 'turtlebot3', close_lanes: [], open_lanes: [20, 21]}"
```

After reopening the lanes, subsequent RMF dispatches and Nav2 graph-based goals can
use that route again.

## License

This project is licensed under the Apache License 2.0. See [LICENSE](LICENSE) for details.