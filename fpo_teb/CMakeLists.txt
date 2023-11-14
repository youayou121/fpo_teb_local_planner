cmake_minimum_required(VERSION 2.8.3)
project(fpo_teb)

# Set to Release in order to speed up the program significantly
set(CMAKE_BUILD_TYPE Release) #None, Debug, Release, RelWithDebInfo, MinSizeRel
find_package(OpenCV 4.2.0 REQUIRED)
## Find catkin macros and libraries
find_package(catkin REQUIRED COMPONENTS
  base_local_planner
  costmap_2d
  costmap_converter
  cmake_modules
  dynamic_reconfigure
  geometry_msgs
  interactive_markers
  message_generation
  nav_core
  nav_msgs
  mbf_costmap_core
  mbf_msgs
  roscpp
  std_msgs
  pluginlib
  tf2
  tf2_eigen
  tf2_geometry_msgs
  tf2_ros
  visualization_msgs
  teb_local_planner
  obstacle_prediction
)
message(STATUS "System: ${CMAKE_SYSTEM}")

SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR}/cmake_modules)
message(STATUS "${CMAKE_MODULE_PATH}")
find_package(Boost REQUIRED COMPONENTS system thread graph)
find_package(SUITESPARSE REQUIRED)
find_package(G2O REQUIRED)

if (EXISTS "FindEigen3.cmake")
  find_package(Eigen3 REQUIRED)
  set(Eigen_INCLUDE_DIRS ${Eigen3_INCLUDE_DIRS})
elseif (EXISTS "FindEigen.cmake")
  find_package(Eigen REQUIRED)
elseif (EXISTS "FindEigen.cmake")
  message(WARNING "No findEigen cmake script found. You must provde one of them,
  e.g. by adding it to ${PROJECT_SOURCE_DIR}/cmake_modules.")
endif (EXISTS "FindEigen3.cmake")

set(EXTERNAL_INCLUDE_DIRS ${Eigen_INCLUDE_DIRS} ${SUITESPARSE_INCLUDE_DIRS} ${G2O_INCLUDE_DIR})
set(EXTERNAL_LIBS ${SUITESPARSE_LIBRARIES} ${G2O_LIBRARIES})

IF(NOT MSVC)
  include(CheckCXXCompilerFlag)
  CHECK_CXX_COMPILER_FLAG("-std=c++11" COMPILER_SUPPORTS_CXX11)
  CHECK_CXX_COMPILER_FLAG("-std=c++0x" COMPILER_SUPPORTS_CXX0X)
if(COMPILER_SUPPORTS_CXX11)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
elseif(COMPILER_SUPPORTS_CXX0X)
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++0x")
else()
  message(STATUS "The compiler ${CMAKE_CXX_COMPILER} has no C++11 support which is required 
  by linked third party packages starting from ROS Jade. Ignore this message for ROS Indigo.")
endif()
endif()

catkin_package(
  INCLUDE_DIRS include ${EXTERNAL_INCLUDE_DIRS}
  LIBRARIES fpo_teb ${EXTERNAL_LIBS}
  CATKIN_DEPENDS
	base_local_planner
	costmap_2d
	costmap_converter
	dynamic_reconfigure
	geometry_msgs
	interactive_markers
	message_runtime
	nav_core
	nav_msgs
	pluginlib
	roscpp
  mbf_costmap_core
	std_msgs
  tf2
  tf2_ros
	visualization_msgs
  DEPENDS SUITESPARSE G2O
)


include_directories(include)
include_directories(
  SYSTEM
  ${EXTERNAL_INCLUDE_DIRS}
  ${catkin_INCLUDE_DIRS}
  ${OpenCV2_INCLUDE_DIRS}
)


add_library(fpo_teb
   src/timed_elastic_band.cpp
   src/optimal_planner.cpp
   src/obstacles.cpp
   src/teb_config.cpp
   src/visualization.cpp
   src/recovery_behaviors.cpp
   src/homotopy_class_planner.cpp
   src/teb_local_planner_ros.cpp
   src/graph_search.cpp
)

add_dependencies(fpo_teb ${PROJECT_NAME}_gencfg)

add_dependencies(fpo_teb ${PROJECT_NAME}_generate_messages_cpp)

target_link_libraries(fpo_teb
   ${EXTERNAL_LIBS}
   ${catkin_LIBRARIES}
   ${OpenCV_LIBRARIES}
   
)


install(PROGRAMS
  scripts/cmd_vel_to_ackermann_drive.py
  # scripts/export_to_mat.py
  # scripts/export_to_svg.py
  # scripts/publish_dynamic_obstacle.py
  # scripts/publish_test_obstacles.py
  # scripts/publish_viapoints.py
  # scripts/visualize_velocity_profile.py
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(TARGETS fpo_teb
   LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
)

install(DIRECTORY include/${PROJECT_NAME}/
   DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION}
   #FILES_MATCHING PATTERN "*.h"
   PATTERN ".svn" EXCLUDE
)


install(FILES
  fpo_teb_plugin.xml
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)

install(DIRECTORY
  launch cfg scripts
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
  PATTERN ".svn" EXCLUDE
)

