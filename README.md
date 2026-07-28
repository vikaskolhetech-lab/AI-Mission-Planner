## mission_interfaces/msg/DroneCommand.msg
```
nano src/mission_interfaces/msg/DroneCommand.msg
```
```
string mission

float32 x
float32 y
float32 z

float32 altitude

float32 duration

string pattern

int32 loops
```
## mission_interfaces/CMakeLists.txt
```
nano src/mission_interfaces/CMakeLists.txt
```
```
cmake_minimum_required(VERSION 3.8)
project(mission_interfaces)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)
find_package(std_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(rosidl_default_generators REQUIRED)

rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/DroneCommand.msg"
  DEPENDENCIES
    std_msgs
    geometry_msgs
)

ament_export_dependencies(rosidl_default_runtime)

ament_package()
```
## mission_interfaces/package.xml
```
nano src/mission_interfaces/package.xml
```
```
<?xml version="1.0"?>
<package format="3">

  <name>mission_interfaces</name>
  <version>0.0.0</version>

  <description>Custom interfaces for AI Drone Mission</description>

  <maintainer email="vikas@example.com">Vikas</maintainer>
  <license>Apache-2.0</license>

  <buildtool_depend>ament_cmake</buildtool_depend>

  <build_depend>rosidl_default_generators</build_depend>

  <build_export_depend>std_msgs</build_export_depend>
  <build_export_depend>geometry_msgs</build_export_depend>

  <exec_depend>std_msgs</exec_depend>
  <exec_depend>geometry_msgs</exec_depend>
  <exec_depend>rosidl_default_runtime</exec_depend>

  <member_of_group>rosidl_interface_packages</member_of_group>

  <export>
    <build_type>ament_cmake</build_type>
  </export>

</package>
```
## 2. llm_bridge Package
```
nano src/llm_bridge/package.xml
```
```
<?xml version="1.0"?>
<?xml-model href="http://download.ros.org/schema/package_format3.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
<package format="3">
  <name>llm_bridge</name>
  <version>0.0.0</version>
  <description>Bridge between natural language and drone commands using Ollama</description>
  <maintainer email="vikas@example.com">Vikas</maintainer>
  <license>Apache-2.0</license>

  <depend>rclpy</depend>
  <depend>mission_interfaces</depend>

  <test_depend>ament_copyright</test_depend>
  <test_depend>ament_flake8</test_depend>
  <test_depend>ament_pep257</test_depend>
  <test_depend>python3-pytest</test_depend>

  <export>
    <build_type>ament_python</build_type>
  </export>
</package>
```
## llm_bridge/setup.py
```
nano src/llm_bridge/setup.py
```
```
from setuptools import find_packages, setup

package_name = 'llm_bridge'

setup(
    name=package_name,
    version='0.0.0',
    packages=find_packages(exclude=['test']),
    data_files=[
        ('share/ament_index/resource_index/packages',
            ['resource/' + package_name]),
        ('share/' + package_name, ['package.xml']),
    ],
    install_requires=['setuptools'],
    zip_safe=True,
    maintainer='vikas',
    maintainer_email='vikas@example.com',
    description='Bridge between natural language and drone commands using Ollama',
    license='Apache-2.0',
    extras_require={
        'test': [
            'pytest',
        ],
    },
    entry_points={
        "console_scripts":[
            "bridge = llm_bridge.bridge:main",
        ],
    },
)
```
## llm_bridge/setup.cfg
```
nano src/llm_bridge/setup.cfg
```
```
[develop]
script_dir=$base/lib/llm_bridge
[install]
install_scripts=$base/lib/llm_bridge
```
## llm_bridge/resource/llm_bridge
```
nano src/llm_bridge/resource/llm_bridge
```
```
llm_bridge
```
## llm_bridge/llm_bridge/bridge.py
```
nano src/llm_bridge/llm_bridge/bridge.py
```
```
import json
import requests

import rclpy
from rclpy.node import Node

from mission_interfaces.msg import DroneCommand


class LLMBridge(Node):

    def __init__(self):
        super().__init__("llm_bridge")

        self.publisher = self.create_publisher(
            DroneCommand,
            "/mission/raw",
            10
        )

        self.get_logger().info("LLM Bridge Started")

    def ask_llm(self, command):

        prompt = f"""
Convert the following drone command into ONLY valid JSON.

Command:
{command}

Return ONLY this JSON format.

{{
    "mission":"square",
    "x":0.0,
    "y":0.0,
    "z":-5.0,
    "altitude":5.0,
    "duration":30.0,
    "pattern":"square",
    "loops":1
}}

Return ONLY JSON.
"""

        response = requests.post(
            "http://localhost:11434/api/generate",
            json={
                "model": "llama3.1",
                "prompt": prompt,
                "stream": False,
                "format": "json"
            },
            timeout=120
        )

        response.raise_for_status()

        result = response.json()["response"]

        self.get_logger().info(f"\nLLM Response:\n{result}")

        return json.loads(result)

    def publish(self, mission):

        if mission is None:
            self.get_logger().error("No valid mission received.")
            return

        msg = DroneCommand()

        msg.mission = str(mission.get("mission", "patrol"))

        msg.x = float(mission.get("x", 0.0))
        msg.y = float(mission.get("y", 0.0))
        msg.z = float(mission.get("z", -5.0))

        msg.altitude = float(mission.get("altitude", 5.0))
        msg.duration = float(mission.get("duration", 30.0))

        # Handle pattern safely
        pattern = mission.get("pattern", "square")

        if isinstance(pattern, dict):
            pattern = pattern.get("type", "square")

        msg.pattern = str(pattern)

        msg.loops = int(mission.get("loops", 1))

        self.publisher.publish(msg)

        self.get_logger().info("Mission Published")


def main(args=None):

    rclpy.init(args=args)

    node = LLMBridge()

    command = input("Drone Command: ")

    try:

        mission = node.ask_llm(command)

        node.publish(mission)  # <-- Updated call

    except Exception as e:

        node.get_logger().error(str(e))

    node.destroy_node()

    rclpy.shutdown()


if __name__ == "__main__":
    main()
```
### 3. mission_validator Package
## mission_validator/package.xml
```
nano src/mission_validator/package.xml
```
```
<?xml version="1.0"?>
<package format="3">

  <name>mission_validator</name>
  <version>0.0.0</version>

  <description>Mission validation node</description>

  <maintainer email="vikas@example.com">vikas</maintainer>

  <license>Apache-2.0</license>

  <buildtool_depend>ament_python</buildtool_depend>

  <depend>rclpy</depend>
  <depend>mission_interfaces</depend>

  <test_depend>ament_lint_auto</test_depend>
  <test_depend>ament_lint_common</test_depend>
  <test_depend>python3-pytest</test_depend>

  <export>
    <build_type>ament_python</build_type>
  </export>

</package>
```
## mission_validator/setup.py
```
nano src/mission_validator/setup.py
```
```
from setuptools import find_packages, setup

package_name = "mission_validator"

setup(
    name=package_name,
    version="0.0.0",
    packages=find_packages(exclude=["test"]),
    data_files=[
        (
            "share/ament_index/resource_index/packages",
            ["resource/" + package_name],
        ),
        (
            "share/" + package_name,
            ["package.xml"],
        ),
    ],
    install_requires=["setuptools"],
    zip_safe=True,
    maintainer="vikas",
    maintainer_email="vikas@example.com",
    description="Mission validation node",
    license="Apache-2.0",
    tests_require=["pytest"],
    entry_points={
        "console_scripts": [
            "validator = mission_validator.validator:main",
        ],
    },
)
```
## mission_validator/resource/mission_validator
```
nano src/mission_validator/resource/mission_validator
```
```
mission_validator
```
## mission_validator/mission_validator/validator.py
```
nano src/mission_validator/mission_validator/validator.py
```
```
#!/usr/bin/env python3

import math

import rclpy
from rclpy.node import Node

from mission_interfaces.msg import DroneCommand


class MissionValidator(Node):

    def __init__(self):
        super().__init__("mission_validator")

        self.subscription = self.create_subscription(
            DroneCommand,
            "/mission/raw",
            self.mission_callback,
            10,
        )

        self.publisher = self.create_publisher(
            DroneCommand,
            "/mission/validated",
            10,
        )

        self.get_logger().info("Mission Validator Started")

    def mission_callback(self, msg: DroneCommand):

        if not self.validate(msg):
            return

        self.publisher.publish(msg)
        self.get_logger().info("Mission validated and published.")

    def validate(self, msg: DroneCommand) -> bool:

        valid_commands = [
            "takeoff",
            "goto",
            "square",
            "circle",
            "patrol",
            "land",
        ]

        if msg.mission.lower() not in valid_commands:
            self.get_logger().error(
                f"Invalid mission: {msg.mission}"
            )
            return False

        if msg.altitude < 1.0 or msg.altitude > 20.0:
            self.get_logger().error(
                f"Invalid altitude: {msg.altitude}"
            )
            return False

        if math.isnan(msg.x) or math.isnan(msg.y):
            self.get_logger().error("Invalid coordinates.")
            return False

        if msg.duration < 0:
            self.get_logger().error("Duration must be positive.")
            return False

        if msg.loops < 0:
            self.get_logger().error("Loops must be positive.")
            return False

        return True


def main(args=None):
    rclpy.init(args=args)

    node = MissionValidator()

    try:
        rclpy.spin(node)
    except KeyboardInterrupt:
        pass

    node.destroy_node()
    rclpy.shutdown()


if __name__ == "__main__":
    main()
```
### 4. mission_executor Package 
## mission_executor/package.xml
```
nano src/mission_executor/package.xml
```
```
<?xml version="1.0"?>
<?xml-model href="http://download.ros.org/schema/package_format3.xsd" schematypens="http://www.w3.org/2001/XMLSchema"?>
<package format="3">
  <name>mission_executor</name>
  <version>0.0.0</version>
  <description>Deterministic mission executor for PX4 drone</description>
  <maintainer email="vikas@example.com">Vikas</maintainer>
  <license>Apache-2.0</license>

  <depend>rclpy</depend>
  <depend>px4_msgs</depend>
  <depend>geometry_msgs</depend>
  <depend>std_msgs</depend>
  <depend>mission_interfaces</depend>   <!-- ✅ ADDED -->

  <test_depend>ament_copyright</test_depend>
  <test_depend>ament_flake8</test_depend>
  <test_depend>ament_pep257</test_depend>
  <test_depend>python3-pytest</test_depend>

  <export>
    <build_type>ament_python</build_type>
  </export>
</package>
```
## mission_executor/setup.py
```
nano src/mission_executor/setup.py
```
```
from setuptools import find_packages, setup

package_name = 'mission_executor'

setup(
    name=package_name,
    version='0.0.0',
    packages=find_packages(exclude=['test']),
    data_files=[
        (
            'share/ament_index/resource_index/packages',
            ['resource/' + package_name]
        ),
        (
            'share/' + package_name,
            ['package.xml']
        ),
    ],
    install_requires=['setuptools'],
    zip_safe=True,
    maintainer='vikas',
    maintainer_email='vikas@example.com',
    description='Deterministic mission executor for PX4 drone',
    license='Apache-2.0',
    tests_require=['pytest'],
    entry_points={
        'console_scripts': [
            'mission_executor = mission_executor.executor:main',
        ],
    },
)
```
## mission_executor/resource/mission_executor
```
nano src/mission_executor/resource/mission_executor
```
```
mission_executor
```
## mission_executor/mission_executor/executor.py
```
nano src/mission_executor/mission_executor/executor.py
```
```
import rclpy
from rclpy.node import Node
import time

from mission_interfaces.msg import DroneCommand

from px4_msgs.msg import (
    OffboardControlMode,
    TrajectorySetpoint,
    VehicleCommand,
    VehicleLocalPosition
)


class MissionExecutor(Node):

    def __init__(self):

        super().__init__("mission_executor")

        self.get_logger().info(
            "Autonomous Mission Executor Started"
        )

        self.mission = None

        self.waypoints = []
        self.current_wp = 0
        self.mission_active = False

        # current position for navigation
        self.current_position = [0.0, 0.0, 0.0]

        # offboard mode and arming timers
        self.offboard_counter = 0
        self.offboard_sent = False
        self.arm_sent = False

        # time‑based waypoint holding
        self.wp_start_time = None
        self.wp_hold_time = 5.0

        self.timer = self.create_timer(
            0.1,
            self.control_loop
        )

        self.create_subscription(
            DroneCommand,
            "/mission/validated",
            self.mission_callback,
            10
        )

        self.create_subscription(
            VehicleLocalPosition,
            "/fmu/out/vehicle_local_position_v1",
            self.position_callback,
            10
        )

        self.offboard_pub = self.create_publisher(
            OffboardControlMode,
            "/fmu/in/offboard_control_mode",
            10
        )

        self.setpoint_pub = self.create_publisher(
            TrajectorySetpoint,
            "/fmu/in/trajectory_setpoint",
            10
        )

        self.command_pub = self.create_publisher(
            VehicleCommand,
            "/fmu/in/vehicle_command",
            10
        )

    def timestamp(self):
        return int(self.get_clock().now().nanoseconds / 1000)

    def mission_callback(self, msg):
        self.get_logger().info(f"Mission received: {msg.pattern}")

        # square mission loading
        if msg.pattern == "square":
            self.waypoints = [
                [0.0, 0.0, -5.0],
                [5.0, 0.0, -5.0],
                [5.0, 5.0, -5.0],
                [0.0, 5.0, -5.0],
                [0.0, 0.0, -5.0]
            ]

            self.current_wp = 0
            self.mission_active = True
            self.wp_start_time = time.time()

            self.get_logger().info("Square mission loaded")

    def position_callback(self, msg):
        self.current_position = [msg.x, msg.y, msg.z]

    def publish_offboard(self):
        msg = OffboardControlMode()
        msg.position = True
        msg.velocity = False
        msg.acceleration = False
        msg.attitude = False
        msg.body_rate = False
        msg.timestamp = self.timestamp()
        self.offboard_pub.publish(msg)

    def publish_waypoint(self):
        if not self.mission_active:
            return
        point = self.waypoints[self.current_wp]
        msg = TrajectorySetpoint()
        msg.position = point
        msg.yaw = 0.0
        msg.timestamp = self.timestamp()
        self.setpoint_pub.publish(msg)

    def send_command(self, cmd):
        msg = VehicleCommand()
        msg.command = cmd
        msg.param1 = 1.0
        msg.param2 = 0.0
        if cmd == VehicleCommand.VEHICLE_CMD_DO_SET_MODE:
            msg.param2 = 6.0  # OFFBOARD mode
        msg.target_system = 1
        msg.target_component = 1
        msg.source_system = 1
        msg.source_component = 1
        msg.from_external = True
        msg.timestamp = self.timestamp()
        self.command_pub.publish(msg)

    def control_loop(self):
        self.publish_offboard()

        if not self.mission_active:
            return

        self.publish_waypoint()

        self.offboard_counter += 1

        # OFFBOARD mode request after 2 seconds
        if self.offboard_counter == 20:
            self.get_logger().info("Requesting OFFBOARD mode")
            self.send_command(VehicleCommand.VEHICLE_CMD_DO_SET_MODE)

        # Arm after 4 seconds
        if self.offboard_counter == 40:
            self.get_logger().info("Arming drone")
            self.send_command(VehicleCommand.VEHICLE_CMD_COMPONENT_ARM_DISARM)

        # time‑based waypoint progression
        if self.wp_start_time is None:
            self.wp_start_time = time.time()

        elapsed = time.time() - self.wp_start_time

        if elapsed > self.wp_hold_time:
            if self.current_wp < len(self.waypoints) - 1:
                self.current_wp += 1
                self.wp_start_time = time.time()
                self.get_logger().info(f"Moving to waypoint {self.current_wp}")
            else:
                self.mission_active = False
                self.get_logger().info("Square mission completed")


def main(args=None):
    rclpy.init(args=args)
    node = MissionExecutor()
    rclpy.spin(node)
    node.destroy_node()
    rclpy.shutdown()


if __name__ == "__main__":
    main()
```
###  Build & Run Instructions
## 1. Build the workspace
```
cd ~/px4_ros_ws
colcon build --packages-select mission_interfaces llm_bridge mission_validator mission_executor
source install/setup.bash

```
## Terminal 1
```
cd ~/drone_sim/PX4-Autopilot && make px4_sitl gz_x500
```
## Terminal 2
```
MicroXRCEAgent udp4 -p 8888
```
## Terminal 3
```
source /opt/ros/jazzy/setup.bash && source ~/px4_ros_ws/install/setup.bash && ros2 run mission_validator validator
```
## Terminal 4
```
source /opt/ros/jazzy/setup.bash && source ~/px4_ros_ws/install/setup.bash && ros2 run mission_executor mission_executor
```
## Terminal 5
```
source /opt/ros/jazzy/setup.bash && source ~/px4_ros_ws/install/setup.bash && ros2 run llm_bridge bridge
```
In Terminal 6, type your command, e.g.:
```
Drone Command: Take off to 5 meters and fly a square patrol.
```

