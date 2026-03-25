cmake_minimum_required(VERSION 3.8)
project(ratslam)

if(POLICY CMP0072)
  cmake_policy(SET CMP0072 NEW)
endif()

# Default to C99
if(NOT CMAKE_C_STANDARD)
  set(CMAKE_C_STANDARD 99)
endif()

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${PROJECT_SOURCE_DIR}/cmake)

# find dependencies
find_package(ament_cmake REQUIRED)
find_package(rclcpp REQUIRED)
find_package(topological_msgs REQUIRED)
find_package(sensor_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(nav_msgs REQUIRED)
find_package(visualization_msgs REQUIRED)
find_package(cv_bridge REQUIRED)

find_package(Boost REQUIRED COMPONENTS serialization)
find_package(Irrlicht REQUIRED)
set(OpenGL_GL_PREFERENCE GLVND)
find_package(OpenGL REQUIRED)

find_package(OpenCV REQUIRED)
set(ament_dependencies
  rclcpp
  topological_msgs
  sensor_msgs
  geometry_msgs
  nav_msgs
  visualization_msgs
  cv_bridge
)

# Ratslam library
add_library(ratslam 
  src/ratslam/local_view_match.cpp 
  src/ratslam/posecell_network.cpp 
  src/ratslam/experience_map.cpp 
  src/ratslam/visual_odometry.cpp
)

target_include_directories(ratslam PUBLIC
  $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
  $<INSTALL_INTERFACE:include>
  ${Boost_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}
  ${IRRLICHT_INCLUDE_DIRS}
)

target_link_libraries(ratslam
  ${Boost_LIBRARIES}
  ${OpenCV_LIBS}
)

# ratslam_lv executable
add_executable(ratslam_lv src/main_lv.cpp)
ament_target_dependencies(ratslam_lv ${ament_dependencies})
target_include_directories(ratslam_lv PUBLIC
  ${CMAKE_CURRENT_SOURCE_DIR}/src
  ${Boost_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}
  ${IRRLICHT_INCLUDE_DIRS}
)
target_link_libraries(ratslam_lv
  ratslam
  ${IRRLICHT_LIBRARIES}
  ${OPENGL_LIBRARIES}
  ${OpenCV_LIBS}
  ${Boost_LIBRARIES}
)

# ratslam_pc executable
add_executable(ratslam_pc src/main_pc.cpp)
ament_target_dependencies(ratslam_pc ${ament_dependencies})
target_include_directories(ratslam_pc PUBLIC
  ${CMAKE_CURRENT_SOURCE_DIR}/src
  ${Boost_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}
  ${IRRLICHT_INCLUDE_DIRS}
)
target_link_libraries(ratslam_pc
  ratslam
  ${IRRLICHT_LIBRARIES}
  ${OPENGL_LIBRARIES}
  ${OpenCV_LIBS}
  ${Boost_LIBRARIES}
)

# ratslam_em executable
add_executable(ratslam_em src/main_em.cpp)
ament_target_dependencies(ratslam_em ${ament_dependencies})
target_include_directories(ratslam_em PUBLIC
  ${CMAKE_CURRENT_SOURCE_DIR}/src
  ${Boost_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}
  ${IRRLICHT_INCLUDE_DIRS}
)
target_link_libraries(ratslam_em
  ratslam
  ${IRRLICHT_LIBRARIES}
  ${OPENGL_LIBRARIES}
  ${OpenCV_LIBS}
  ${Boost_LIBRARIES}
)

# ratslam_vo executable
add_executable(ratslam_vo src/main_vo.cpp)
ament_target_dependencies(ratslam_vo ${ament_dependencies})
target_include_directories(ratslam_vo PUBLIC
  ${CMAKE_CURRENT_SOURCE_DIR}/src
  ${Boost_INCLUDE_DIRS}
  ${OpenCV_INCLUDE_DIRS}
  ${IRRLICHT_INCLUDE_DIRS}
)
target_link_libraries(ratslam_vo
  ratslam
  ${IRRLICHT_LIBRARIES}
  ${OPENGL_LIBRARIES}
  ${OpenCV_LIBS}
  ${Boost_LIBRARIES}
)

# Install binaries
install(TARGETS 
  ratslam
  ratslam_lv
  ratslam_pc
  ratslam_em
  ratslam_vo
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION lib/${PROJECT_NAME}
)

install(DIRECTORY
  src/ratslam/
  DESTINATION include/ratslam
  FILES_MATCHING PATTERN "*.h"
)

install(DIRECTORY
  src/graphics/
  DESTINATION include/graphics
  FILES_MATCHING PATTERN "*.h"
)

# Install launch, config, and media directories
install(DIRECTORY 
  launch
  config
  media
  DESTINATION share/${PROJECT_NAME}
)

ament_package()
