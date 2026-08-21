# Build a Complete ROS 2 Swarm Drone Simulation in Gazebo

You are going to act as the **lead ROS 2 robotics engineer** responsible for implementing this project from scratch.

I want you to **actually create the complete project files and code**, not merely explain how I could implement them.

The project is a **4-drone swarm simulation for autonomous area coverage/mapping using ROS 2 Humble and Ignition Gazebo**.

For this first version, **DO NOT use PX4, MAVROS, MAVLink, ArduPilot, or any flight controller**. Those will be integrated later.

The current goal is to build and thoroughly test the swarm intelligence, communication, task allocation, area partitioning, autonomous movement, coverage planning, and visualization entirely using ROS 2 + Gazebo.

---

# 1. PROJECT OBJECTIVE

Build a simulated swarm consisting of:

* 4 drones by default
* Drone ID 0 = Leader
* Drone IDs 1, 2, 3 = Followers

The number of drones must be configurable from a YAML configuration file.

The swarm is given a rectangular area that needs to be covered.

The leader:

1. Knows the total mapping/coverage area.
2. Divides the area into approximately equal-area regions.
3. Assigns one region to every drone.
4. Communicates the assignments through ROS 2.
5. Monitors the status of all drones.
6. Also performs coverage of its own assigned region.

The followers:

1. Receive their assigned region from the leader.
2. Generate a coverage path for that region.
3. Autonomously fly through the region.
4. Report their state/progress to the leader.
5. Stop/land when their assigned region is complete.

The final result should visually demonstrate:

```text
                         LEADER
                           D0
                           |
             +-------------+-------------+
             |             |             |
             v             v             v
            D1            D2            D3
         follower      follower       follower
```

with all four drones communicating through ROS 2.

---

# 2. IMPORTANT SCOPE RESTRICTION

For this version:

## DO NOT implement:

* PX4
* MAVROS
* MAVLink
* ArduPilot
* PX4 SITL
* flight-controller firmware
* real-world flight dynamics
* SLAM
* LiDAR
* depth camera
* point clouds
* complicated mesh-based drone models

These are future extensions.

The current project should be:

```text
ROS 2 Humble
      |
      v
Swarm Intelligence
      |
      v
ROS 2 communication
      |
      v
Gazebo simulation
```

The drone should be controlled directly from ROS 2/Gazebo.

Design the control layer so that PX4/MAVROS can be added later without rewriting the swarm-management logic.

---

# 3. SOFTWARE ENVIRONMENT

Target environment:

* Ubuntu
* ROS 2 Humble
* Ignition Gazebo / Gazebo compatible with the installed ROS 2 Humble environment
* Python 3
* ROS 2 Python nodes
* URDF/Xacro
* YAML configuration

Before writing implementation code:

1. Inspect the environment.
2. Determine the installed Gazebo/Ignition version.
3. Determine the available ROS-Gazebo integration packages.
4. Determine the correct bridge/control mechanism for the installed version.
5. Adapt the implementation to the actual installed environment.

Do NOT blindly assume a Gazebo version.

If a package/API differs between Gazebo versions, use the version actually installed on the machine.

---

# 4. DEVELOPMENT PHILOSOPHY

Build the project incrementally.

Do NOT create a giant untested codebase and assume it works.

The implementation should follow this order:

### Phase 1

Create one basic drone model and successfully spawn it in Gazebo.

### Phase 2

Make one drone move autonomously using ROS 2.

### Phase 3

Spawn multiple instances of the same drone.

### Phase 4

Give every drone a unique ID and namespace.

### Phase 5

Implement leader/follower ROS 2 communication.

### Phase 6

Implement area partitioning.

### Phase 7

Implement waypoint/coverage generation.

### Phase 8

Make all drones autonomously cover their assigned regions.

### Phase 9

Add MarkerArray visualization.

### Phase 10

Add task monitoring and testing.

At every phase:

* build
* run
* verify
* fix errors
* only then proceed

If something fails, diagnose and correct it instead of moving forward with a broken dependency.

---

# 5. DRONE MODEL

I do NOT have an existing drone model.

Create the drone entirely from basic geometric primitives.

Use only:

* cylinders
* cuboids/boxes
* optionally spheres where useful

DO NOT use:

* mesh files
* STL
* DAE
* OBJ
* external drone models

The drone should visually resemble a simple quadrotor.

For example:

```text
             motor          motor
               O------------O
                \          /
                 \  BODY   /
                 /         \
                /           \
               O-------------O
             motor          motor
```

The propellers/fans should be represented using simple geometry if necessary, but:

## FAN ROTATION MUST BE OFF.

Do not implement rotating propellers.

The model only needs to visually represent a quadrotor.

Include:

* visual geometry
* collision geometry
* inertial properties
* mass
* links
* joints
* base frame
* camera link
* control interface

Keep the physical model simple and stable.

---

# 6. XACRO/URDF STRUCTURE

I specifically want the drone description separated into:

```text
robot.urdf.xacro
bot.xacro
camera.xacro
control.xacro
```

The conceptual structure must be:

```text
robot.urdf.xacro
       |
       +---- bot.xacro
       |
       +---- camera.xacro
       |
       +---- control.xacro
```

`robot.urdf.xacro` should include the other three files.

If technically appropriate, also generate a processed:

```text
robot.urdf
```

from the Xacro.

Do not duplicate the drone definition unnecessarily.

---

# 7. bot.xacro

`bot.xacro` should contain the physical drone model.

It should define:

* base link
* drone body
* four arms
* four motor/cylinder representations
* optional simple propeller geometry
* collision geometry
* inertial properties

Use basic geometric primitives only.

No meshes.

No rotating propellers.

Make dimensions and mass configurable using Xacro properties wherever reasonable.

---

# 8. camera.xacro

Each drone should have one simulated RGB camera.

There must be:

* no LiDAR
* no depth camera
* no depth sensor
* no point cloud sensor

The camera should be mounted on the drone and publish a ROS 2 image topic.

The camera is included now because it will be useful for future mapping/detection work.

The first coverage algorithm does NOT need to use the camera image.

---

# 9. control.xacro

Create the appropriate Gazebo/ROS 2 control interface for the current simulation.

The control system must allow a ROS 2 node to command the simulated drone's movement.

Do not introduce PX4/MAVROS.

The exact implementation should depend on the installed Gazebo version.

The controller should expose a clean interface to the higher-level swarm code.

The higher-level swarm code should conceptually see:

```text
DroneController
    |
    +---- takeoff()
    +---- move_to(x, y, z)
    +---- land()
    +---- stop()
```

The actual Gazebo-specific implementation should remain inside the low-level control layer.

This separation is important because PX4/MAVROS may replace this layer later.

---

# 10. MULTI-DRONE ARCHITECTURE

The number of drones must come from YAML.

Default:

```yaml
swarm:
  num_drones: 4
  leader_id: 0
```

For 4 drones:

```text
drone_0 = leader
drone_1 = follower
drone_2 = follower
drone_3 = follower
```

Do not hard-code exactly four drones throughout the software.

The architecture should support:

```text
num_drones = 3
num_drones = 4
num_drones = 5
num_drones = 6
...
```

without rewriting the main swarm logic.

---

# 11. DRONE IDs AND NAMESPACES

Every drone must have a unique ID.

Use namespaces such as:

```text
/drone_0
/drone_1
/drone_2
/drone_3
```

Each drone should have its own:

* pose/state
* command interface
* camera topic
* status topic
* assigned region
* coverage progress

Example:

```text
/drone_0/state
/drone_1/state
/drone_2/state
/drone_3/state
```

and:

```text
/drone_0/camera/image_raw
/drone_1/camera/image_raw
...
```

The implementation must avoid topic collisions.

---

# 12. TAKEOFF BEHAVIOR

All drones should logically begin from the **same takeoff point**.

The takeoff point must be configurable in YAML.

For example:

```yaml
takeoff:
  x: 0.0
  y: 0.0
  z: 0.0
  flight_altitude: 5.0
```

The intended behavior is:

```text
All drones
     |
     v
same takeoff location
     |
     v
take off
     |
     v
move toward assigned regions
     |
     v
perform coverage
```

Because physically spawning several solid bodies at exactly the same pose may cause Gazebo collisions/instability, implement a safe mechanism if necessary.

For example:

* sequential takeoff
* controlled startup
* temporary spawn offsets
* configurable initialization handling

But the **logical mission takeoff point must remain the same**.

Do not silently change the project requirement to permanently different starting locations.

---

# 13. LEADER DRONE

Drone 0 is the leader.

The leader is NOT just a stationary coordinator.

It is also an active mapping/coverage drone.

Therefore:

```text
Drone 0:
    task allocation
    +
    communication
    +
    monitoring
    +
    own area coverage
```

The leader should:

1. Initialize.
2. Determine the mapping area.
3. Calculate the number of regions.
4. Divide the area.
5. Assign regions to all drones, including itself.
6. Publish assignments.
7. Monitor drone status.
8. Start coverage.
9. Continue covering its own region.
10. Detect completion.

---

# 14. FOLLOWER DRONES

Follower nodes should:

1. Initialize.
2. Identify their unique ID.
3. Wait for leader assignment.
4. Receive region information.
5. Confirm assignment.
6. Generate/receive coverage waypoints.
7. Take off.
8. Move to their region.
9. Execute coverage path.
10. Publish progress.
11. Report completion.
12. Stop/land.

---

# 15. ROS 2 COMMUNICATION

Communication must be implemented using actual ROS 2 communication.

Use:

* topics
* services where appropriate
* messages

Do NOT simply call Python functions between leader and follower classes.

The project should demonstrate real distributed ROS 2 communication.

Suggested communication architecture:

```text
                 LEADER
                   |
       +-----------+-----------+
       |           |           |
       v           v           v
 region_assignment/status   commands
       |           |           |
       v           v           v
      D1          D2          D3
```

Possible topics:

```text
/swarm/task_assignment
/swarm/region_assignment
/swarm/status
/swarm/mission_state
/swarm/coverage
```

You may choose a better topic structure if technically appropriate.

Use custom ROS 2 messages if they make the architecture cleaner.

---

# 16. REGION ASSIGNMENT

The total coverage area must be configurable through YAML.

Example:

```yaml
mapping_area:
  min_x: -10.0
  max_x: 10.0
  min_y: -10.0
  max_y: 10.0
  flight_altitude: 5.0
```

The leader must automatically calculate regions.

For four drones, a sensible partition is:

```text
+-------------+-------------+
|             |             |
|     D0      |     D1      |
|   Region 0  |   Region 1  |
|             |             |
+-------------+-------------+
|             |             |
|     D2      |     D3      |
|   Region 2  |   Region 3  |
|             |             |
+-------------+-------------+
```

Each region should have approximately equal area.

Do not hard-code the boundaries.

---

# 17. GENERALIZATION TO DIFFERENT DRONE COUNTS

The partitioning algorithm must work for different values of:

```yaml
num_drones
```

For example:

```text
4 drones → 2 × 2
6 drones → approximately 2 × 3
9 drones → 3 × 3
8 drones → approximately 2 × 4
```

Choose a grid factorization that produces regions as close to square as possible.

For prime numbers where an exact rectangular factorization is impossible, implement a reasonable strategy for dividing the area into approximately equal-area strips/cells.

The key requirement is:

> Every drone should receive approximately the same area.

Do not assume the number of drones will always be four.

---

# 18. COVERAGE ALGORITHM

The first version is **area coverage**, not full SLAM.

Do not implement SLAM.

The drone must cover its assigned region using a systematic coverage path.

Use a lawnmower/boustrophedon-style path:

```text
→ → → → → → →
              ↓
← ← ← ← ← ← ←
              ↓
→ → → → → → →
              ↓
← ← ← ← ← ← ←
```

The coverage path must:

* remain inside the assigned region
* avoid leaving its region
* cover the region systematically
* use configurable spacing
* produce waypoints
* be deterministic
* work for different region sizes

YAML should contain something like:

```yaml
coverage:
  waypoint_spacing: 1.0
  boundary_margin: 0.5
```

---

# 19. EQUAL COVERAGE

The objective is not merely to divide the area equally.

The drones should also receive approximately equal **coverage workload**.

Therefore consider:

* region area
* coverage path length
* waypoint count
* boundary margins

The first implementation can use equal-area partitioning, with approximately equal waypoint density.

Do not attempt an unnecessarily complex optimization algorithm unless needed.

Keep the algorithm understandable and robust.

---

# 20. MARKERARRAY — IMPORTANT

The correct requirement is **MarkerArray**, NOT "mapper array".

Use:

```text
visualization_msgs/MarkerArray
```

for visualization/debugging.

The MarkerArray should show:

### Drone identity

Each drone should have a visible ID:

```text
D0
D1
D2
D3
```

### Role

Show:

```text
D0 → LEADER
D1 → FOLLOWER
D2 → FOLLOWER
D3 → FOLLOWER
```

### Region

Show the assigned region boundaries.

For example:

```text
Region 0 → Drone 0
Region 1 → Drone 1
Region 2 → Drone 2
Region 3 → Drone 3
```

### Current position

Display each drone's current position.

### Coverage

Visualize the coverage progress where practical.

### Waypoints

Display the generated coverage waypoints/path.

Every drone/region must have deterministic Marker IDs.

Do not accidentally reuse marker IDs between drones.

---

# 21. TESTING INDIVIDUAL DRONES

A major requirement is that each drone can be tested independently using its ID.

For example:

```text
test drone 0
test drone 1
test drone 2
test drone 3
```

The software should make it possible to:

* select a specific drone
* inspect its assigned region
* inspect its state
* inspect its path
* verify its ID
* verify its communication
* verify its coverage

Ideally expose a configuration such as:

```yaml
testing:
  enabled: false
  test_drone_id: 0
```

When enabled, the system should allow testing of an individual drone without destroying the rest of the architecture.

---

# 22. TASK MONITOR

Create a task-monitoring ROS 2 node.

It should display/log information such as:

```text
Swarm initialized
Number of drones: 4

Leader: Drone 0

Drone 0 → Region 0
Drone 1 → Region 1
Drone 2 → Region 2
Drone 3 → Region 3

Drone 0 → TAKEOFF
Drone 1 → TAKEOFF
Drone 2 → TAKEOFF
Drone 3 → TAKEOFF

Drone 0 → COVERING
Drone 1 → COVERING
Drone 2 → COVERING
Drone 3 → COVERING

Drone 1 → COMPLETE
Drone 2 → COMPLETE
Drone 3 → COMPLETE
Drone 0 → COMPLETE

SWARM MISSION COMPLETE
```

The monitor should be able to detect:

* initialization
* assignment
* takeoff
* movement
* coverage
* completion
* errors/timeouts

---

# 23. YAML CONFIGURATION

Create a central YAML configuration file.

It should contain at least:

```yaml
swarm:
  num_drones: 4
  leader_id: 0

mapping_area:
  min_x: -10.0
  max_x: 10.0
  min_y: -10.0
  max_y: 10.0
  flight_altitude: 5.0

takeoff:
  x: 0.0
  y: 0.0
  z: 0.0

coverage:
  waypoint_spacing: 1.0
  boundary_margin: 0.5

simulation:
  use_sim_time: true

testing:
  enabled: false
  test_drone_id: 0
```

Add any other parameters that are necessary.

Do not scatter hard-coded parameters throughout Python files.

---

# 24. PROJECT STRUCTURE

Create a complete ROS 2 package with a clean structure similar to:

```text
swarm_drone/
│
├── package.xml
├── setup.py
├── setup.cfg
├── resource/
│   └── swarm_drone
│
├── swarm_drone/
│   ├── __init__.py
│   ├── leader.py
│   ├── follower.py
│   ├── drone_controller.py
│   ├── task_manager.py
│   ├── coverage_planner.py
│   ├── region_allocator.py
│   ├── marker_manager.py
│   └── task_monitor.py
│
├── launch/
│   ├── simulation.launch.py
│   ├── auto.launch.py
│   └── teleop.launch.py
│
├── config/
│   └── swarm.yaml
│
├── urdf/
│   ├── robot.urdf.xacro
│   ├── bot.xacro
│   ├── camera.xacro
│   └── control.xacro
│
├── worlds/
│   └── mapping_world.sdf
│
├── models/
│   └── ...
│
├── rviz/
│   └── swarm.rviz
│
└── README.md
```

If additional files are required, add them.

---

# 25. LAUNCH FILES

Create these launch files:

## simulation.launch.py

Responsible for:

* starting Gazebo
* loading the world
* spawning the configured number of drones
* setting namespaces
* loading robot descriptions
* starting required bridges/controllers

## auto.launch.py

Responsible for launching the autonomous swarm mission.

It should start:

* leader
* followers
* task manager
* coverage planner
* marker manager
* task monitor

## teleop.launch.py

Provide a simple way to manually test/control a selected drone.

This is mainly for debugging.

---

# 26. GAZEBO WORLD

Create a simple mapping world.

The world should contain:

* flat ground
* clearly defined coverage area
* enough space for the swarm
* optional simple boundary markers/walls

Do not make the world visually complicated.

The purpose is to test the swarm algorithm.

---

# 27. DRONE SPAWNING

The launch system must spawn the configured number of drones automatically.

Do NOT copy-paste four separate drone definitions.

Use a loop/parameterized launch mechanism.

Conceptually:

```text
num_drones = 4

for id in range(num_drones):
    spawn drone_id
```

Every spawned drone should receive:

* unique name
* unique namespace
* unique ID
* role
* appropriate topics

---

# 28. HIGH-LEVEL SOFTWARE ARCHITECTURE

Keep responsibilities separated.

Recommended:

```text
                    TASK MANAGER
                         |
                         v
                 REGION ALLOCATOR
                         |
                         v
                      LEADER
                         |
             +-----------+-----------+
             |           |           |
             v           v           v
            D1          D2          D3
             |           |           |
             +-----------+-----------+
                         |
                         v
                 COVERAGE PLANNER
                         |
                         v
                  DRONE CONTROLLER
                         |
                         v
                       GAZEBO
```

Do not put the entire project inside one Python file.

---

# 29. SEPARATE HIGH-LEVEL AND LOW-LEVEL LOGIC

This is extremely important for future PX4/MAVROS integration.

The following should NOT depend directly on Gazebo:

* region allocation
* task allocation
* leader logic
* follower logic
* coverage planning
* MarkerArray logic
* task monitoring

Only the low-level controller should know how the simulated drone actually moves.

For example:

```text
coverage_planner.py
        |
        v
waypoints
        |
        v
drone_controller.py
        |
        v
Gazebo
```

Later:

```text
coverage_planner.py
        |
        v
waypoints
        |
        v
drone_controller.py
        |
        v
MAVROS
        |
        v
PX4
```

This is intentional.

---

# 30. STATE MACHINE

Implement a clear drone state machine.

For example:

```text
INITIALIZING
     |
     v
WAITING_FOR_TASK
     |
     v
TASK_ASSIGNED
     |
     v
TAKING_OFF
     |
     v
GOING_TO_REGION
     |
     v
COVERING
     |
     v
COMPLETED
     |
     v
LANDING
     |
     v
IDLE
```

Handle failures/timeouts sensibly.

---

# 31. LEADER STATE MACHINE

The leader should have states such as:

```text
INITIALIZING
     |
     v
WAITING_FOR_SWARM
     |
     v
ALLOCATING_TASKS
     |
     v
WAITING_FOR_ACK
     |
     v
MISSION_RUNNING
     |
     v
MONITORING
     |
     v
MISSION_COMPLETE
```

The leader should not begin mission execution until the swarm is ready.

---

# 32. COMMUNICATION HANDSHAKE

Implement a basic handshake.

For example:

```text
Leader
   |
   | "Are you ready?"
   |
   +----> D1
   +----> D2
   +----> D3
             |
             | "READY"
             v
          Leader
             |
             | assign regions
             v
        all drones
```

Each drone should acknowledge its region assignment.

This prevents the leader from assuming a task was received when it wasn't.

---

# 33. COLLISION AVOIDANCE

The first implementation does not need a sophisticated swarm collision-avoidance algorithm.

However, the design must avoid obvious collisions.

Since all drones begin at the same logical takeoff point:

* use controlled/sequential takeoff if necessary
* ensure drones move away from the takeoff point safely
* ensure assigned regions do not overlap
* ensure coverage paths stay inside their own regions

Do not introduce complex formation control unless necessary.

---

# 34. COMPLETION CONDITION

A drone should be considered complete when:

```text
all of its coverage waypoints have been visited
```

The leader should consider the entire mission complete when:

```text
all drones report COMPLETE
```

Then:

```text
MISSION COMPLETE
```

should be published/logged.

---

# 35. VISUALIZATION

Create an RViz configuration.

Show:

* Gazebo/world frame if available
* drone positions
* drone IDs
* region boundaries
* waypoints
* coverage progress
* MarkerArray visualization

Use clear marker namespaces.

Example:

```text
drone_markers
region_markers
waypoint_markers
coverage_markers
```

---

# 36. ERROR HANDLING

The code should gracefully handle:

* missing YAML parameters
* invalid drone IDs
* invalid number of drones
* region allocation failures
* missing leader
* follower not responding
* command timeout
* Gazebo connection problems
* invalid coverage area
* zero/negative area
* impossible configuration

Do not let the whole system crash because one follower reports an error.

The leader should be able to detect failed/unresponsive drones.

For this first version, simply report the failure clearly rather than implementing sophisticated fault recovery.

---

# 37. TESTING REQUIREMENTS

Create tests where practical.

At minimum, test the pure Python algorithms independently of Gazebo:

### Region allocation

Verify:

* correct number of regions
* no overlapping regions
* approximately equal area
* all regions inside the requested area

### Coverage planner

Verify:

* waypoints remain inside assigned region
* waypoint spacing is respected
* coverage path is generated
* different region sizes work

### Drone IDs

Verify:

```text
0 → leader
1 → follower
2 → follower
3 → follower
```

### YAML

Verify that changing:

```yaml
num_drones: 4
```

to:

```yaml
num_drones: 6
```

changes the swarm configuration correctly.

---

# 38. README

Create a comprehensive README containing:

## Installation

Every dependency.

## Build

Exact commands.

Example:

```bash
cd ~/your_workspace
colcon build --symlink-install
source install/setup.bash
```

## Run Gazebo

Exact command.

## Run simulation

Exact command.

## Run autonomous swarm

Exact command.

## Run RViz

Exact command.

## Test one drone

Exact command.

## Change number of drones

Explain which YAML parameter to edit.

## Change mapping area

Explain which YAML parameters to edit.

## Troubleshooting

Include common errors and fixes.

---

# 39. TERMINAL COMMANDS

At the end, provide an exact step-by-step procedure starting from an empty ROS 2 workspace.

For example:

```bash
mkdir -p ~/swarm_ws/src
cd ~/swarm_ws/src
```

Then create the package and files.

Then:

```bash
cd ~/swarm_ws
colcon build --symlink-install
source install/setup.bash
```

Then commands to launch the system.

Do not leave steps implicit.

---

# 40. DO NOT FABRICATE APIs

This is very important.

If a Gazebo plugin, ROS 2 package, message type, or command differs based on the installed version:

1. inspect the environment
2. determine the actual API
3. implement against the installed API

Do not invent package names or commands.

If a required dependency is unavailable, explain exactly what is missing and provide the appropriate installation command.

---

# 41. DEBUGGING MODE

When I give you an error from the terminal:

1. Identify the exact cause.
2. Explain it briefly.
3. Tell me exactly which file to modify.
4. Give me the replacement code or exact edit.
5. Give me the command to rebuild.
6. Give me the command to rerun.
7. Check for likely secondary errors.

Do not restart the entire project unnecessarily.

Do not tell me "it should work" without verification.

---

# 42. IMPORTANT: DO NOT OVERENGINEER

The goal is a working academic/research prototype.

Do not add unnecessary:

* machine learning
* SLAM
* reinforcement learning
* complex physics
* sophisticated planners
* unnecessary dependencies
* external drone models
* Docker
* PX4
* MAVROS

unless they become necessary.

Keep the implementation understandable enough that I can study and modify it.

---

# 43. FUTURE EXTENSIBILITY

Design the project so that these can be added later:

```text
Current:
ROS 2 → Gazebo

Future:
ROS 2 → MAVROS → PX4 → Gazebo
```

Potential future additions:

* PX4
* MAVROS
* MAVLink
* actual autonomous flight dynamics
* SLAM
* LiDAR
* depth camera
* object detection
* real mapping
* more advanced task allocation
* dynamic region reassignment
* fault tolerance
* real hardware

But DO NOT implement these now.

---

# 44. FINAL EXPECTED RESULT

When everything is working, I should be able to run the simulation and see:

```text
             GAZEBO

                 D0
              LEADER
                 |
        +--------+--------+
        |        |        |
        v        v        v
       D1       D2       D3
```

Then:

```text
       TOTAL COVERAGE AREA

┌─────────────────┬─────────────────┐
│                 │                 │
│       D0        │       D1        │
│    Region 0     │    Region 1     │
│                 │                 │
├─────────────────┼─────────────────┤
│                 │                 │
│       D2        │       D3        │
│    Region 2     │    Region 3     │
│                 │                 │
└─────────────────┴─────────────────┘
```

The drones should:

1. initialize
2. communicate
3. take off from the same logical takeoff point
4. move toward their assigned regions
5. execute coverage paths
6. remain inside their assigned regions
7. publish status
8. visualize IDs and regions through MarkerArray
9. finish when coverage is complete

The leader should simultaneously:

* allocate tasks
* communicate with followers
* monitor them
* cover its own region

---

# 45. HOW I WANT YOU TO WORK

Do NOT respond with only an architecture diagram.

Actually implement the project.

Start by:

### STEP 1

Inspect the environment and determine:

* Ubuntu version
* ROS 2 Humble installation
* Gazebo/Ignition version
* available ROS-Gazebo bridge/control packages
* Python version

### STEP 2

Create the workspace/package structure.

### STEP 3

Create the drone Xacro files.

### STEP 4

Create the Gazebo world.

### STEP 5

Make ONE drone spawn and move.

### STEP 6

Verify it.

### STEP 7

Expand to configurable multi-drone spawning.

### STEP 8

Implement IDs/namespaces.

### STEP 9

Implement leader/follower communication.

### STEP 10

Implement region allocation.

### STEP 11

Implement coverage planning.

### STEP 12

Implement autonomous swarm coverage.

### STEP 13

Implement MarkerArray visualization.

### STEP 14

Implement task monitoring.

### STEP 15

Add tests.

### STEP 16

Create the README and exact run instructions.

At every stage, ensure the previous stage works before building on it.

If you need to make a technical assumption, state it clearly and choose the simplest robust solution.

If there is a genuine ambiguity that prevents implementation, ask me. Otherwise, **make a reasonable engineering decision and continue rather than repeatedly asking for confirmation.**

The final project must be a real, runnable ROS 2 Humble package rather than pseudocode.

