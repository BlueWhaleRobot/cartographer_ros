# Copyright 2016 The Cartographer Authors
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

cmake_minimum_required(VERSION 2.8)

project(cartographer_ros)

set(PACKAGE_DEPENDENCIES
  eigen_conversions
  geometry_msgs
  cartographer_ros_msgs
  roscpp
  rosbag
  sensor_msgs
  nav_msgs
  tf2
  tf2_eigen
  tf2_ros
  urdf
)

set(CMAKE_CXX_FLAGS "-pthread -std=c++11 -Wreorder")

if(NOT CMAKE_BUILD_TYPE OR CMAKE_BUILD_TYPE STREQUAL "")
  set(CMAKE_BUILD_TYPE Release)
endif()

if(CMAKE_BUILD_TYPE STREQUAL "Release")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3 -DNDEBUG")
elseif(CMAKE_BUILD_TYPE STREQUAL "RelWithDebInfo")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3 -g -DNDEBUG")
elseif(CMAKE_BUILD_TYPE STREQUAL "Debug")
  message(FATAL_ERROR "Cartographer is too slow to be useful in debug mode.")
else()
  message(FATAL_ERROR "Unknown CMAKE_BUILD_TYPE: ${CMAKE_BUILD_TYPE}")
endif()

message(STATUS "Build type: ${CMAKE_BUILD_TYPE}")

find_package(catkin REQUIRED COMPONENTS ${PACKAGE_DEPENDENCIES})
include_directories(${catkin_INCLUDE_DIRS})

catkin_package(
  CATKIN_DEPENDS
    message_runtime
    ${PACKAGE_DEPENDENCIES}
)

LIST(APPEND CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake_modules)

find_package(PCL REQUIRED COMPONENTS common io)
include_directories(${PCL_INCLUDE_DIRS})
link_directories(${PCL_LIBRARY_DIRS})
add_definitions(${PCL_DEFINITIONS})

find_package(Eigen3 REQUIRED)
include_directories(${EIGEN_INCLUDE_DIRS})

find_package(cartographer REQUIRED)
include_directories(${CARTOGRAPHER_INCLUDE_DIRS})
link_directories(${CARTOGRAPHER_LIBRARY_DIRS})

find_package(ZLIB REQUIRED)
include_directories(${ZLIB_INCLUDE_DIRS})

find_package(Boost REQUIRED
 COMPONENTS
 system
 iostreams
)
add_definitions(${BOOST_DEFINITIONS})
include_directories(${Boost_INCLUDE_DIRS}
${PROJECT_SOURCE_DIR}
)
link_directories(${Boost_LIBRARY_DIRS})

add_executable(cartographer_node
  cartographer_ros/node_main.cc
  cartographer_ros/node.cc
  cartographer_ros/msg_conversion.cc
  cartographer_ros/ros_log_sink.cc
  cartographer_ros/sensor_bridge.cc
  cartographer_ros/time_conversion.cc
  cartographer_ros/tf_bridge.cc
  cartographer_ros/map_builder_bridge.cc
  cartographer_ros/assets_writer.cc
  cartographer_ros/occupancy_grid.cc
  cartographer_ros/node_options.cc
  cartographer_ros/map_writer.cc
)
target_link_libraries(cartographer_node
  ${CARTOGRAPHER_LIBRARIES}
  ${PCL_LIBRARIES}
  ${catkin_LIBRARIES}
  gflags
  yaml-cpp
)
add_dependencies(cartographer_node
  ${catkin_EXPORTED_TARGETS}
)

catkin_add_gtest(time_conversion_test
  cartographer_ros/time_conversion_test.cc
  cartographer_ros/time_conversion.cc
)
target_link_libraries(time_conversion_test
  ${CARTOGRAPHER_LIBRARIES}
  ${GTEST_BOTH_LIBRARIES}
  ${catkin_LIBRARIES}
)
add_dependencies(time_conversion_test
  ${catkin_EXPORTED_TARGETS}
)

add_executable(cartographer_assets_writer
  cartographer_ros/assets_writer_main.cc
  cartographer_ros/bag_reader.cc
  cartographer_ros/urdf_reader.cc
  cartographer_ros/node.cc
  cartographer_ros/msg_conversion.cc
  cartographer_ros/ros_log_sink.cc
  cartographer_ros/sensor_bridge.cc
  cartographer_ros/time_conversion.cc
  cartographer_ros/tf_bridge.cc
  cartographer_ros/map_builder_bridge.cc
  cartographer_ros/assets_writer.cc
  cartographer_ros/occupancy_grid.cc
  cartographer_ros/node_options.cc
  cartographer_ros/map_writer.cc
)

target_link_libraries(cartographer_assets_writer
  ${CARTOGRAPHER_LIBRARIES}
  ${PCL_LIBRARIES}
  ${catkin_LIBRARIES}
  yaml-cpp
)

add_executable(cartographer_offline_node
  cartographer_ros/offline_node_main.cc
  cartographer_ros/bag_reader.cc
  cartographer_ros/urdf_reader.cc
  cartographer_ros/node.cc
  cartographer_ros/msg_conversion.cc
  cartographer_ros/ros_log_sink.cc
  cartographer_ros/sensor_bridge.cc
  cartographer_ros/time_conversion.cc
  cartographer_ros/tf_bridge.cc
  cartographer_ros/map_builder_bridge.cc
  cartographer_ros/assets_writer.cc
  cartographer_ros/occupancy_grid.cc
  cartographer_ros/node_options.cc
  cartographer_ros/map_writer.cc
)

target_link_libraries(cartographer_offline_node
  ${CARTOGRAPHER_LIBRARIES}
  ${PCL_LIBRARIES}
  ${catkin_LIBRARIES}
  yaml-cpp
)

install(DIRECTORY launch urdf configuration_files
  DESTINATION ${CATKIN_PACKAGE_SHARE_DESTINATION}
)

install(TARGETS cartographer_node
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(PROGRAMS scripts/tf_remove_frames.py
  DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(TARGETS cartographer_assets_writer
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)

install(TARGETS cartographer_offline_node
  ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
  RUNTIME DESTINATION ${CATKIN_PACKAGE_BIN_DESTINATION}
)
