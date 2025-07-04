#
# Copyright 2024 Bernd Pfrommer <bernd.pfrommer@gmail.com>
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

cmake_minimum_required(VERSION 3.16)
project(apriltag_detector_mit)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic -Werror)
endif()
set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(ament_cmake_ros REQUIRED)
find_package(apriltag_detector REQUIRED)
find_package(apriltag_mit REQUIRED)
find_package(apriltag_msgs REQUIRED)
find_package(cv_bridge REQUIRED)
find_package(image_transport REQUIRED)
find_package(pluginlib REQUIRED)
find_package(rclcpp REQUIRED)
find_package(rclcpp_components REQUIRED)
find_package(sensor_msgs REQUIRED)

set(ament_dependencies
  apriltag_detector::apriltag_detector
  ${apriltag_msgs_TARGETS}
  cv_bridge::cv_bridge
  image_transport::image_transport
  pluginlib::pluginlib
  rclcpp::rclcpp
  rclcpp_components::component
  ${sensor_msgs_TARGETS})

#
# --------- plugin library (exported)
#
pluginlib_export_plugin_description_file(apriltag_detector plugins.xml)

add_library(${PROJECT_NAME} SHARED src/detector.cpp)

target_link_libraries(${PROJECT_NAME} ${ament_dependencies} apriltag_mit::apriltag_mit)

target_include_directories(
  ${PROJECT_NAME} PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:include>)

ament_export_targets(${PROJECT_NAME}_export HAS_LIBRARY_TARGET)
ament_export_libraries(${PROJECT_NAME})

# the shared library goes into the global lib dir so it can be used as a
# composable node by other projects

install(
  TARGETS ${PROJECT_NAME}
  EXPORT ${PROJECT_NAME}_export)

if(BUILD_TESTING)
  find_package(ament_cmake REQUIRED)
  find_package(ament_cmake_copyright REQUIRED)
  find_package(ament_cmake_cppcheck REQUIRED)
  find_package(ament_cmake_cpplint REQUIRED)
  find_package(ament_cmake_clang_format REQUIRED)
  find_package(ament_cmake_flake8 REQUIRED)
  find_package(ament_cmake_lint_cmake REQUIRED)
  find_package(ament_cmake_pep257 REQUIRED)
  find_package(ament_cmake_xmllint REQUIRED)

  ament_copyright()
  ament_cppcheck(LANGUAGE c++)
  ament_cpplint(FILTERS "-build/include,-runtime/indentation_namespace")
  ament_clang_format(CONFIG_FILE .clang-format)
  ament_flake8()
  ament_lint_cmake()
  ament_pep257()
  ament_xmllint()
endif()

ament_package()
