cmake_minimum_required(VERSION 3.5)
project(topological_msgs)

find_package(ament_cmake REQUIRED)
find_package(rosidl_default_generators REQUIRED)
find_package(std_msgs REQUIRED)
find_package(geometry_msgs REQUIRED)
find_package(builtin_interfaces REQUIRED)

rosidl_generate_interfaces(${PROJECT_NAME}
  "msg/ViewTemplate.msg"
  "msg/TopologicalNode.msg"
  "msg/TopologicalEdge.msg"
  "msg/TopologicalMap.msg"
  "msg/TopologicalAction.msg"
  DEPENDENCIES std_msgs geometry_msgs builtin_interfaces
)

ament_export_dependencies(rosidl_default_runtime)

install(DIRECTORY msg DESTINATION share/${PROJECT_NAME})

ament_package()
