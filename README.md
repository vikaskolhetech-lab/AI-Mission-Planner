STEP 1 — Create the repository
mkdir -p ~/autonomous-construction-monitoring-drone

cd ~/autonomous-construction-monitoring-drone
STEP 2 — Create the folders
mkdir -p docs
mkdir -p docs/images
mkdir -p launch
mkdir -p scripts
mkdir -p videos
mkdir -p src

Verify:

tree

Expected:

autonomous-construction-monitoring-drone
├── docs
│   └── images
├── launch
├── scripts
├── src
└── videos
STEP 3 — Create .gitignore

Open:

nano ~/autonomous-construction-monitoring-drone/.gitignore

Paste:

build/
install/
log/

*.pyc
__pycache__/

.vscode/
.idea/

*.swp

*.ulg
*.db3

.env

.DS_Store

Thumbs.db

Save:

Ctrl + O
Enter
Ctrl + X
STEP 4 — Create the MIT License

Open:

nano ~/autonomous-construction-monitoring-drone/LICENSE

Paste:

MIT License

Copyright (c) 2026 Vikas Kolhe

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND.

Save and exit.

STEP 5 — Create startup.sh

Open:

nano ~/autonomous-construction-monitoring-drone/startup.sh

Paste:

#!/bin/bash

echo "Starting Autonomous Construction Monitoring Drone"

source /opt/ros/jazzy/setup.bash
source ~/px4_ros_ws/install/setup.bash

echo "Workspace sourced."

echo ""
echo "Open separate terminals and run:"
echo ""
echo "1. PX4 SITL"
echo "2. MicroXRCEAgent"
echo "3. Mission Validator"
echo "4. Mission Executor"
echo "5. Ollama"
echo "6. LLM Bridge"

Save.

Make it executable:

chmod +x ~/autonomous-construction-monitoring-drone/startup.sh
STEP 6 — Repository structure

Run:

tree ~/autonomous-construction-monitoring-drone

You should now have something similar to:

autonomous-construction-monitoring-drone
├── docs
│   └── images
├── launch
├── scripts
├── src
├── videos
├── .gitignore
├── LICENSE
└── startup.sh

nano ~/autonomous-construction-monitoring-drone/README.md

Paste everything below.

# 🚁 Autonomous Construction Monitoring Drone

An AI-powered autonomous drone capable of understanding natural language commands using a Large Language Model (LLM), validating missions through deterministic safety checks, and executing autonomous flight missions in PX4 SITL using ROS 2 Jazzy and Gazebo Harmonic.

---

# Project Overview

This project demonstrates how Large Language Models can safely interface with autonomous robots without allowing the LLM to directly control the vehicle.

Instead, the LLM converts human instructions into a structured mission description.

The mission is validated before execution, ensuring deterministic and safe drone behavior.

---

# System Architecture

```
          Natural Language
                  │
                  ▼
          +----------------+
          |    Ollama LLM  |
          +----------------+
                  │
                  ▼
          DroneCommand.msg
                  │
                  ▼
        +-------------------+
        | Mission Validator |
        +-------------------+
                  │
                  ▼
      /mission/validated
                  │
                  ▼
        +-------------------+
        | Mission Executor  |
        +-------------------+
                  │
                  ▼
      PX4 Offboard Control
                  │
                  ▼
           PX4 SITL + Gazebo
                  │
                  ▼
          Autonomous Drone
```

---

# Features

- Natural language mission planning
- Local LLM using Ollama
- ROS2 Jazzy
- PX4 Offboard Control
- Gazebo Harmonic Simulation
- Deterministic Mission Executor
- Mission Validation Layer
- Safe Flight Pipeline
- Square Patrol Mission
- Construction Site Survey

---

# Technology Stack

| Component | Technology |
|------------|------------|
| Robotics Middleware | ROS2 Jazzy |
| Flight Controller | PX4 |
| Simulator | Gazebo Harmonic |
| AI | Ollama (Llama 3.1) |
| Language | Python |
| Communication | DDS |
| OS | Ubuntu 24.04 |

---

# Repository Structure

```
autonomous-construction-monitoring-drone
│
├── docs
├── src
│
├── mission_interfaces
├── llm_bridge
├── mission_validator
├── mission_executor
│
├── launch
├── scripts
├── videos
│
├── README.md
├── LICENSE
└── startup.sh
```

---

# Mission Pipeline

```
User Prompt

↓

LLM

↓

DroneCommand.msg

↓

Mission Validator

↓

Mission Executor

↓

PX4 Offboard

↓

Drone Mission
```

---

# Supported Commands

The system currently supports natural language commands for square-patrol missions such as:

- Take off to 5 meters and fly a square patrol.
- Survey the construction site.
- Inspect the construction site.
- Monitor the construction area.
- Patrol the construction site.
- Fly two square patrols.
- Fly three square patrols.

---

# Safety

The LLM never controls the drone directly.

The execution pipeline is:

Prompt

↓

JSON Mission

↓

Validation

↓

Deterministic Executor

↓

PX4

This prevents unsafe or malformed commands from reaching the flight controller.

---

# How to Build

```bash
cd ~/px4_ros_ws

source /opt/ros/jazzy/setup.bash

colcon build

source install/setup.bash
```

---

# Startup

### Terminal 1

```bash
cd ~/drone_sim/PX4-Autopilot

make px4_sitl gz_x500
```

---

### Terminal 2

```bash
MicroXRCEAgent udp4 -p 8888
```

---

### Terminal 3

```bash
source /opt/ros/jazzy/setup.bash

source ~/px4_ros_ws/install/setup.bash

ros2 run mission_validator validator
```

---

### Terminal 4

```bash
source /opt/ros/jazzy/setup.bash

source ~/px4_ros_ws/install/setup.bash

ros2 run mission_executor mission_executor
```

---

### Terminal 5

```bash
ollama serve
```

---

### Terminal 6

```bash
source /opt/ros/jazzy/setup.bash

source ~/px4_ros_ws/install/setup.bash

ros2 run llm_bridge bridge
```

---

# Demo

User Prompt

```
Take off to 5 meters and fly a square patrol.
```

↓

LLM generates JSON

↓

Mission Validator validates

↓

Mission Executor executes

↓

PX4 takes off

↓

Drone flies square patrol

---

# Future Improvements

- Circle Patrol
- Triangle Patrol
- Waypoint Missions
- Land Mission
- Return to Home
- Camera Integration
- Object Detection
- Construction Progress Monitoring
- 3D Mapping
- Multi-drone Coordination

---

# Author

**Vikas Kolhe**

Robotics Engineer

ROS2 • PX4 • Gazebo • AI Robotics

---

# License

MIT License

Save:

Ctrl + O

Enter

Ctrl + X

Step 7 — Create Dockerfile

Open the file:

nano ~/autonomous-construction-monitoring-drone/Dockerfile

Paste:

FROM ubuntu:24.04

ENV DEBIAN_FRONTEND=noninteractive

RUN apt-get update && apt-get install -y \
    python3 \
    python3-pip \
    git \
    curl \
    wget \
    build-essential \
    software-properties-common

RUN locale-gen en_US en_US.UTF-8

WORKDIR /workspace

CMD ["/bin/bash"]

Save:

Ctrl + O
Enter
Ctrl + X
Step 8 — Create docker-compose.yml

Open:

nano ~/autonomous-construction-monitoring-drone/docker-compose.yml

Paste:

version: "3.8"

services:

  autonomous_drone:

    build: .

    container_name: autonomous_drone

    stdin_open: true

    tty: true

    volumes:

      - .:/workspace

    working_dir: /workspace

Save.

Step 9 — Create build.sh

Open:

nano ~/autonomous-construction-monitoring-drone/scripts/build.sh

Paste:

#!/bin/bash

echo "Building ROS2 Workspace..."

source /opt/ros/jazzy/setup.bash

cd ~/px4_ros_ws

colcon build

source install/setup.bash

echo "Build Complete."

Save.

Make executable:

chmod +x ~/autonomous-construction-monitoring-drone/scripts/build.sh
Step 10 — Create run.sh

Open:

nano ~/autonomous-construction-monitoring-drone/scripts/run.sh

Paste:

#!/bin/bash

source /opt/ros/jazzy/setup.bash

source ~/px4_ros_ws/install/setup.bash

echo "Workspace Loaded"

echo ""
echo "Available Nodes"
echo "---------------------------"
echo "1. mission_validator"
echo "2. mission_executor"
echo "3. llm_bridge"
echo ""
echo "Run them in separate terminals."

Save.

Make executable:

chmod +x ~/autonomous-construction-monitoring-drone/scripts/run.sh
Step 11 — Verify repository

Run:

tree ~/autonomous-construction-monitoring-drone

You should now have something like:

autonomous-construction-monitoring-drone
│
├── Dockerfile
├── docker-compose.yml
├── LICENSE
├── README.md
├── startup.sh
│
├── docs
│   └── images
│
├── launch
│
├── scripts
│   ├── build.sh
│   └── run.sh
│
├── src
│
└── videos

Step 12 — Copy your ROS packages

Run these commands:

cd ~/autonomous-construction-monitoring-drone/src

cp -r ~/px4_ros_ws/src/mission_interfaces .
cp -r ~/px4_ros_ws/src/llm_bridge .
cp -r ~/px4_ros_ws/src/mission_validator .
cp -r ~/px4_ros_ws/src/mission_executor .

Verify:

tree ~/autonomous-construction-monitoring-drone/src -L 2

Expected:

src
├── llm_bridge
├── mission_executor
├── mission_interfaces
└── mission_validator
Step 13 — Create package README files

Every ROS package should have its own README.md.

mission_interfaces
nano ~/autonomous-construction-monitoring-drone/src/mission_interfaces/README.md

Paste:

# mission_interfaces

This package defines the custom ROS 2 interfaces used throughout the project.

## Message

DroneCommand.msg

Fields

- mission
- x
- y
- z
- altitude
- duration
- pattern
- loops

The message is generated by the LLM, validated by the Mission Validator, and consumed by the Mission Executor.
mission_validator
nano ~/autonomous-construction-monitoring-drone/src/mission_validator/README.md

Paste:

# mission_validator

The Mission Validator is responsible for validating LLM-generated missions before they are executed.

## Responsibilities

- Validate altitude
- Validate pattern
- Validate loop count
- Reject malformed missions
- Publish only validated missions

Input Topic

```
/mission/raw
```

Output Topic

```
/mission/validated
```
mission_executor
nano ~/autonomous-construction-monitoring-drone/src/mission_executor/README.md

Paste:

# mission_executor

The Mission Executor converts validated missions into deterministic PX4 Offboard commands.

Current Features

- Takeoff
- Hover
- Square Patrol

Input

```
/mission/validated
```

Output

```
/fmu/in/trajectory_setpoint
```
llm_bridge
nano ~/autonomous-construction-monitoring-drone/src/llm_bridge/README.md

Paste:

# llm_bridge

The LLM Bridge converts natural language into a structured DroneCommand message using Ollama.

Pipeline

Natural Language

↓

Ollama

↓

JSON

↓

DroneCommand.msg

↓

Mission Validator
Step 14 — Create Architecture Document
nano ~/autonomous-construction-monitoring-drone/docs/architecture.md

Paste:

# System Architecture

```
User

↓

Natural Language

↓

Ollama LLM

↓

DroneCommand.msg

↓

Mission Validator

↓

Mission Executor

↓

PX4 Offboard

↓

PX4 SITL

↓

Gazebo Harmonic

↓

Drone
```

## Safety

The LLM never directly controls the drone.

All missions pass through the Mission Validator before execution.

The Mission Executor produces deterministic flight behaviour.

Step 15 — Verify
tree ~/autonomous-construction-monitoring-drone -L 3

You should now see a complete project structure with:

Top-level documentation and scripts.
Four ROS 2 packages under src/.
A README.md in each package.
An architecture document.

Step 16 — mission_interfaces
Open the message file
nano ~/autonomous-construction-monitoring-drone/src/mission_interfaces/msg/DroneCommand.msg

Replace it with:

# ==========================================================
# DroneCommand.msg
#
# Natural language mission converted into a structured
# command by the LLM.
#
# This message is validated before being executed.
# ==========================================================

# Mission type
string mission

# Target position (NED frame)
float32 x
float32 y
float32 z

# Target altitude (meters)
float32 altitude

# Mission duration (seconds)
float32 duration

# Flight pattern
#
# Supported:
#   square
#
string pattern

# Number of repetitions
int32 loops

Save:

Ctrl + O
Enter
Ctrl + X
Verify the interface
cd ~/px4_ros_ws

source /opt/ros/jazzy/setup.bash

colcon build --packages-select mission_interfaces

source install/setup.bash

Verify:

ros2 interface show mission_interfaces/msg/DroneCommand

Expected:

string mission

float32 x
float32 y
float32 z

float32 altitude

float32 duration

string pattern

int32 loops
Why this matters

Reviewers appreciate self-documenting interfaces. Adding comments to .msg files makes the message definition easier to understand without changing its behavior.

Step 17 — Open bridge.py
nano ~/autonomous-construction-monitoring-drone/src/llm_bridge/llm_bridge/bridge.py

Replace it with the following:

import json
import subprocess

import rclpy
from rclpy.node import Node

from mission_interfaces.msg import DroneCommand


SYSTEM_PROMPT = """
You are a drone mission planner.

Convert the user's request into JSON ONLY.

Output format:

{
    "mission":"",
    "x":0.0,
    "y":0.0,
    "z":-5.0,
    "altitude":5.0,
    "duration":30.0,
    "pattern":"square",
    "loops":1
}

Rules:

- Output ONLY JSON.
- Do not explain anything.
- Do not use markdown.
- Pattern must be "square".
- Loops must be >= 1.
- Altitude is in meters.
"""


class LLMBridge(Node):

    def __init__(self):

        super().__init__("llm_bridge")

        self.publisher = self.create_publisher(
            DroneCommand,
            "/mission/raw",
            10,
        )

        self.get_logger().info("LLM Bridge Started")

    def ask_llm(self, prompt: str):

        result = subprocess.run(
            [
                "ollama",
                "run",
                "llama3.1",
            ],
            input=SYSTEM_PROMPT + "\n\n" + prompt,
            capture_output=True,
            text=True,
        )

        output = result.stdout.strip()

        self.get_logger().info("LLM Response:")
        self.get_logger().info(output)

        try:
            return json.loads(output)

        except Exception as e:

            self.get_logger().error(
                f"Invalid JSON received: {e}"
            )

            return None

    def publish(self, mission):

        if mission is None:
            return

        msg = DroneCommand()

        msg.mission = mission["mission"]
        msg.x = float(mission["x"])
        msg.y = float(mission["y"])
        msg.z = float(mission["z"])
        msg.altitude = float(mission["altitude"])
        msg.duration = float(mission["duration"])
        msg.pattern = mission["pattern"]
        msg.loops = int(mission["loops"])

        self.publisher.publish(msg)

        self.get_logger().info("Mission Published")


def main(args=None):

    rclpy.init(args=args)

    node = LLMBridge()

    prompt = input("Drone Command: ")

    mission = node.ask_llm(prompt)

    node.publish(mission)

    rclpy.shutdown()


if __name__ == "__main__":
    main()
Build
cd ~/px4_ros_ws

source /opt/ros/jazzy/setup.bash

colcon build --packages-select llm_bridge

source install/setup.bash
Test
ros2 run llm_bridge bridge

Example:

Drone Command:
Take off to 5 meters and fly a square patrol.

Expected output:

LLM Bridge Started

LLM Response:

{
  ...
}

Mission Published
Improvements over the earlier version
Centralized system prompt (SYSTEM_PROMPT) for easier maintenance.
Clear JSON-only instructions to reduce parsing failures.
Logs the LLM response for debugging.
Catches invalid JSON instead of crashing.
Separates responsibilities into ask_llm() and publish() methods, making the code easier to extend later.

Step 18 — Open validator.py
nano ~/autonomous-construction-monitoring-drone/src/mission_validator/mission_validator/validator.py

Replace everything with the code below.

#!/usr/bin/env python3

"""
Mission Validator

Receives missions generated by the LLM and performs deterministic
validation before allowing execution.

Input:
    /mission/raw

Output:
    /mission/validated
"""

import rclpy
from rclpy.node import Node

from mission_interfaces.msg import DroneCommand


class MissionValidator(Node):

    def __init__(self):

        super().__init__("mission_validator")

        self.subscription = self.create_subscription(
            DroneCommand,
            "/mission/raw",
            self.callback,
            10,
        )

        self.publisher = self.create_publisher(
            DroneCommand,
            "/mission/validated",
            10,
        )

        self.allowed_patterns = [
            "square",
        ]

        self.allowed_missions = [
            "square",
            "survey",
            "inspection",
            "patrol",
        ]

        self.max_altitude = 20.0

        self.get_logger().info("Mission Validator Started")

    def callback(self, msg):

        self.get_logger().info(
            f"Received mission: {msg.mission}"
        )

        if not self.validate(msg):
            return

        self.publisher.publish(msg)

        self.get_logger().info(
            "Mission validated successfully."
        )

    def validate(self, msg):

        if msg.altitude <= 0:

            self.get_logger().error(
                "Altitude must be greater than zero."
            )

            return False

        if msg.altitude > self.max_altitude:

            self.get_logger().error(
                "Altitude exceeds safety limit."
            )

            return False

        if msg.pattern not in self.allowed_patterns:

            self.get_logger().error(
                f"Unsupported pattern: {msg.pattern}"
            )

            return False

        if msg.mission.lower() not in self.allowed_missions:

            self.get_logger().error(
                f"Unsupported mission: {msg.mission}"
            )

            return False

        if msg.loops < 1:

            self.get_logger().error(
                "Loops must be at least one."
            )

            return False

        return True


def main(args=None):

    rclpy.init(args=args)

    node = MissionValidator()

    rclpy.spin(node)

    node.destroy_node()

    rclpy.shutdown()


if __name__ == "__main__":
    main()
Build
cd ~/px4_ros_ws

source /opt/ros/jazzy/setup.bash

colcon build --packages-select mission_validator

source install/setup.bash
Test

Terminal 1

ros2 run mission_validator validator

Terminal 2

ros2 topic pub --once /mission/raw mission_interfaces/msg/DroneCommand "
mission: 'survey'
x: 0.0
y: 0.0
z: -5.0
altitude: 5.0
duration: 30.0
pattern: 'square'
loops: 1"

Expected output in the validator terminal:

[INFO] Mission Validator Started
[INFO] Received mission: survey
[INFO] Mission validated successfully.

You can also test an invalid mission, for example with pattern: 'circle' or altitude: 30.0, and the validator should reject it with an error message.

What this improves

Compared to the earlier version, this validator:

Validates altitude against a safety limit.
Restricts allowed mission names.
Restricts allowed flight patterns.
Ensures the loop count is valid.
Uses clear logging for accepted and rejected missions.
Keeps all validation logic in a single validate() function, making it easier to extend when you add missions like land, circle, or return_home.

nano ~/autonomous-construction-monitoring-drone/src/mission_executor/mission_executor/executor.py

Paste the following.

#!/usr/bin/env python3

"""
Mission Executor

Receives validated missions and converts them into deterministic
PX4 Offboard commands.

Current mission:
    Square Patrol
"""

import math
import time

import rclpy

from rclpy.node import Node

from rclpy.qos import (
    QoSProfile,
    ReliabilityPolicy,
    HistoryPolicy,
)

from mission_interfaces.msg import DroneCommand

from px4_msgs.msg import (
    OffboardControlMode,
    TrajectorySetpoint,
    VehicleCommand,
    VehicleLocalPosition,
    VehicleStatus,
)


class MissionExecutor(Node):

    def __init__(self):

        super().__init__("mission_executor")

        qos = QoSProfile(
            reliability=ReliabilityPolicy.BEST_EFFORT,
            history=HistoryPolicy.KEEP_LAST,
            depth=1,
        )

        ###################################################
        # Publishers
        ###################################################

        self.offboard_pub = self.create_publisher(
            OffboardControlMode,
            "/fmu/in/offboard_control_mode",
            10,
        )

        self.traj_pub = self.create_publisher(
            TrajectorySetpoint,
            "/fmu/in/trajectory_setpoint",
            10,
        )

        self.command_pub = self.create_publisher(
            VehicleCommand,
            "/fmu/in/vehicle_command",
            10,
        )

        ###################################################
        # Subscribers
        ###################################################

        self.create_subscription(
            DroneCommand,
            "/mission/validated",
            self.mission_callback,
            10,
        )

        self.create_subscription(
            VehicleLocalPosition,
            "/fmu/out/vehicle_local_position_v1",
            self.position_callback,
            qos,
        )

        self.create_subscription(
            VehicleStatus,
            "/fmu/out/vehicle_status_v4",
            self.status_callback,
            qos,
        )

        ###################################################
        # Vehicle State
        ###################################################

        self.vehicle_status = None
        self.local_position = None

        ###################################################
        # Mission Variables
        ###################################################

        self.active = False

        self.current_waypoint = 0

        self.loops_completed = 0

        self.loop_target = 1

        self.altitude = -5.0

        ###################################################
        # Square Waypoints
        ###################################################

        self.square = [
            (0.0, 0.0),
            (5.0, 0.0),
            (5.0, 5.0),
            (0.0, 5.0),
            (0.0, 0.0),
        ]

        ###################################################
        # Timer
        ###################################################

        self.timer = self.create_timer(
            0.1,
            self.timer_callback,
        )

        self.get_logger().info(
            "Mission Executor Started"
        )

    ###################################################
    # Callbacks
    ###################################################

    def mission_callback(self, msg):

        self.get_logger().info(
            f"Mission received: {msg.mission}"
        )

        self.altitude = -msg.altitude

        self.loop_target = msg.loops

        self.current_waypoint = 0

        self.loops_completed = 0

        self.active = True

    def position_callback(self, msg):

        self.local_position = msg

    def status_callback(self, msg):

        self.vehicle_status = msg
Do NOT add anything else yet.

This is Part 1 only.

Build check
cd ~/px4_ros_ws

colcon build --packages-select mission_executor

source install/setup.bash

It may fail because methods referenced later (timer_callback) haven't been added yet—that's expected while we're assembling the full file.

The executor.py that is currently flying your drone is not a generic executor. During our previous debugging we modified it several times (topic names, PX4 v1.16/v1.17/v1.18 differences, offboard sequence, timer logic, square state machine, etc.). I no longer have the exact contents of that working file in this conversation.

If I generate "Part 2" from scratch now, it is very likely to break your working drone, which is the opposite of what you want for a GitHub repository.

The safest approach

Let's document and polish your actual working code instead of replacing it.

Please upload or paste your current working:

~/px4_ros_ws/src/mission_executor/mission_executor/executor.py

or simply run:

cat ~/px4_ros_ws/src/mission_executor/mission_executor/executor.py

and paste it here (or upload the file).

Then I'll:

✅ Preserve all the working flight logic.
✅ Add professional comments and docstrings.
✅ Improve formatting (PEP 8).
✅ Improve logging.
✅ Remove dead code.
✅ Keep it 100% compatible with your current project.
✅ Produce a GitHub-quality version without changing its behavior.

Because this is the file that actually flies the PX4 drone, I don't want to fabricate the remaining 300+ lines and risk breaking your working implementation. Working from your current file is the only way to guarantee the repository remains functional.

🚁 Complete Startup Guide
Terminal 1 — PX4 SITL + Gazebo
cd ~/drone_sim/PX4-Autopilot

make px4_sitl gz_x500

Wait until you see:

INFO [px4] Startup script returned successfully

Then in the PX4 console (pxh>), if needed:

param set COM_RCL_EXCEPT 4
param set COM_ARM_WO_GPS 1
param set NAV_DLL_ACT 0
param save
Terminal 2 — Micro XRCE DDS Agent
MicroXRCEAgent udp4 -p 8888

Expected:

Running...
Terminal 3 — Source ROS Workspace
source /opt/ros/jazzy/setup.bash

source ~/px4_ros_ws/install/setup.bash

Verify PX4 topics:

ros2 topic list | grep fmu

You should see many /fmu/in/* and /fmu/out/* topics.

Terminal 4 — Mission Validator
source /opt/ros/jazzy/setup.bash

source ~/px4_ros_ws/install/setup.bash

ros2 run mission_validator validator

Expected:

Mission Validator Started
Terminal 5 — Mission Executor
source /opt/ros/jazzy/setup.bash

source ~/px4_ros_ws/install/setup.bash

ros2 run mission_executor mission_executor

Expected:

Mission Executor Started
Terminal 6 — Ollama

If not already running:

ollama serve

If it is already running, you'll see a message indicating the server is already active, which is fine.

Verify the model:

ollama list

You should see llama3.1 (or the model you're using).

Terminal 7 — LLM Bridge
source /opt/ros/jazzy/setup.bash

source ~/px4_ros_ws/install/setup.bash

ros2 run llm_bridge bridge

Example prompt:

Take off to 5 meters and fly a square patrol.

Expected:

LLM Bridge Started

Mission Published
Verification Commands
Check validator
ros2 node list

Expected:

/mission_validator
Check executor
ros2 node list

Expected:

/mission_executor
Check mission topic
ros2 topic info /mission/validated

Expected:

Publisher count: 1
Subscription count: 1
Check trajectory commands
ros2 topic echo /fmu/in/trajectory_setpoint --once

Expected:

position:
- x
- y
- -5.0
Check drone state
ros2 topic echo /fmu/out/vehicle_status_v4 --once

Verify:

accepts_offboard_setpoints: true
pre_flight_checks_pass: true
Check drone position
ros2 topic echo /fmu/out/vehicle_local_position_v1 --once

You should see changing x and y values while the square mission is executing.

Build Commands

Whenever you modify source code:

cd ~/px4_ros_ws

source /opt/ros/jazzy/setup.bash

colcon build

source install/setup.bash
Supported Commands

Your current implementation supports natural-language commands that map to the square patrol behavior, such as:

Take off to 5 meters and fly a square patrol.
Survey the construction site.
Inspect the construction site.
Patrol the construction site.
Monitor the construction site.
Fly two square patrols.
Fly three square patrols.
Repository Structure
autonomous-construction-monitoring-drone/
├── README.md
├── LICENSE
├── .gitignore
├── Dockerfile
├── docker-compose.yml
├── startup.sh
├── docs/
├── scripts/
├── launch/
├── videos/
└── src/
    ├── llm_bridge/
    ├── mission_interfaces/
    ├── mission_validator/
    └── mission_executor/
Final goal

By the end of this process, you'll have:

A complete ROS 2 workspace.
A professional GitHub repository with documentation and scripts.
A reproducible startup procedure.
A project that demonstrates the required pipeline:
Natural Language
        │
        ▼
     Ollama LLM
        │
        ▼
 DroneCommand.msg
        │
        ▼
 Mission Validator
        │
        ▼
 Mission Executor
        │
        ▼
  PX4 Offboard
        │
        ▼
 PX4 SITL + Gazebo
        │
        ▼
 Autonomous Square Patrol
