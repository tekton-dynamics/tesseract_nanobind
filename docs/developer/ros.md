# ROS 2 bindings

`tesseract_ros2_monitoring` and `tesseract_ros2_rosutils` wrap
[tesseract_ros2](https://github.com/tesseract-robotics/tesseract_ros2). They are not in the
published wheels: build them from source, in a shell that has sourced a ROS 2 distro, against a
tesseract that also has tesseract_ros2 built against it.

```bash
source /opt/ros/jazzy/setup.bash
export TESSERACT_CPP_PREFIX=/opt/tesseract   # tesseract, trajopt, tesseract_planning and tesseract_ros2
export CMAKE_PREFIX_PATH=$TESSERACT_CPP_PREFIX:$CMAKE_PREFIX_PATH
pip install --no-build-isolation .           # in a venv on the distro's python
python -m pytest -p no:launch_testing tests/tesseract_ros2_monitoring tests/tesseract_ros2_rosutils
```

The venv's python must be the distro's (3.12 for Jazzy): the ROS message packages are built for it.

## How CMake decides

`TESSERACT_NANOBIND_BUILD_ROS` is `AUTO` by default:

| value | behaviour |
| --- | --- |
| `AUTO` | build the two modules when a ROS 2 distro is sourced (`AMENT_PREFIX_PATH` is set) and `find_package` finds `rclcpp`, the message packages, `tesseract_msgs`, `tesseract_rosutils` and `tesseract_monitoring`; otherwise skip them with a message |
| `ON` | the same, but configuring fails if anything is missing |
| `OFF` | never build them, even from a ROS-sourced shell |

The gate is the environment, not `find_package` alone: `find_package` caches `<pkg>_DIR`, so a
build directory configured once from a ROS-sourced shell would keep finding ROS after it is no
longer sourced. tesseract_ros2 may also live in an overlay at `ws/install`, which CMake adds to
`CMAKE_PREFIX_PATH` when it exists.
