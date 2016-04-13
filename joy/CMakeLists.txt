# http://ros.org/doc/groovy/api/catkin/html/user_guide/supposed.html
cmake_minimum_required(VERSION 2.8.3)
project(joy)

if (DEFINED CATKIN_DEVEL_PREFIX)

# Load catkin and all dependencies required for this package
set(CATKIN_DEPS roscpp diagnostic_updater sensor_msgs)
set(ROSDEP_DEPS joystick)
find_package(catkin REQUIRED ${CATKIN_DEPS})
catkin_package(DEPENDS ${CATKIN_DEPS} ${ROSDEP_DEPS})

# Look for <linux/joystick.h>
include(CheckIncludeFiles)
check_include_files(linux/joystick.h HAVE_LINUX_JOYSTICK_H)

if(HAVE_LINUX_JOYSTICK_H)
  include_directories(msg/cpp ${catkin_INCLUDE_DIRS})
  add_executable(joy_node src/joy_node.cpp)
  target_link_libraries(joy_node ${catkin_LIBRARIES})
else(HAVE_LINUX_JOYSTICK_H)
  message("Warning: no <linux/joystick.h>; won't build joy node")
endif(HAVE_LINUX_JOYSTICK_H)

#catkin_add_nosetests(test/test_joy_msg_migration.py)

# Install targets
install(TARGETS joy_node
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION})

install(DIRECTORY migration_rules
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
  )

else()

message("hello i am in ament")

if(NOT WIN32)
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11 -Wall -Wextra")
endif()

find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rmw_implementation_cmake REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(ros1_shim REQUIRED)

macro(targets)
  if(NOT "${target_suffix} " STREQUAL " ")
    get_rclcpp_information("${rmw_implementation}" "rclcpp${target_suffix}")
  endif()

  add_executable(joy_node${target_suffix} src/joy_node.cpp)
  add_definitions(-DROS2_JOY)
  ament_target_dependencies(joy_node${target_suffix}
    "rclcpp${target_suffix}"
    "sensor_msgs")
  target_link_libraries(joy_node${target_suffix} ros1_shim${target_suffix})

  install(TARGETS
    joy_node${target_suffix}
    ARCHIVE DESTINATION lib
    LIBRARY DESTINATION lib
    RUNTIME DESTINATION bin)
endmacro()

install(
  DIRECTORY include/
  DESTINATION include
)

call_for_each_rmw_implementation(targets GENERATE_DEFAULT)

ament_export_dependencies(rclcpp)
ament_export_dependencies(sensor_msgs)

if(AMENT_ENABLE_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package()

endif()
