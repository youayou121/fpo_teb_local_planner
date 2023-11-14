cmake_minimum_required(VERSION 3.1)
project(teb_local_planner)

# Set to Release in order to speed up the program significantly
set(CMAKE_BUILD_TYPE Release) #None, Debug, Release, RelWithDebInfo, MinSizeRel

## Find catkin macros and libraries
find_package(catkin REQUIRED COMPONENTS
  dynamic_reconfigure
  geometry_msgs
  roscpp
  std_msgs
  costmap_converter
)

add_message_files(
  FILES
  TrajectoryPointMsg.msg
  TrajectoryMsg.msg
  FeedbackMsg.msg
)


## Generate added messages and services with any dependencies listed here
generate_messages(
   DEPENDENCIES
   geometry_msgs std_msgs costmap_converter
)

#add dynamic reconfigure api
#find_package(catkin REQUIRED dynamic_reconfigure)
generate_dynamic_reconfigure_options(
  cfg/TebLocalPlannerReconfigure.cfg
)
catkin_package(
  INCLUDE_DIRS include
)
add_library(${PROJECT_NAME}
  include/teb_local_planner/teb_config.h
  include/teb_local_planner/pose_se2.h
  include/teb_local_planner/obstacles.h
  include/teb_local_planner/misc.h
  include/teb_local_planner/robot_footprint_model.h
  include/teb_local_planner/distance_calculations.h
  src/teb_config.cpp
)
include_directories(include)
include_directories(
  ${EXTERNAL_INCLUDE_DIRS}
  ${catkin_INCLUDE_DIRS}
)


# Dynamic reconfigure: make sure configure headers are built before any node using them
add_dependencies(teb_local_planner ${PROJECT_NAME}_gencfg)
# Generate messages before compiling the lib
add_dependencies(teb_local_planner ${PROJECT_NAME}_generate_messages_cpp)

target_link_libraries(teb_local_planner
   ${EXTERNAL_LIBS}
   ${catkin_LIBRARIES}
)



