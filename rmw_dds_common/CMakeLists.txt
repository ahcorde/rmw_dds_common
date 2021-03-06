cmake_minimum_required(VERSION 3.5)

project(rmw_dds_common)

# Default to C++14
if(NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 14)
endif()

if(CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic)
endif()

find_package(ament_cmake REQUIRED)
find_package(rcpputils REQUIRED)
find_package(rcutils REQUIRED)
find_package(rmw REQUIRED)
find_package(rosidl_default_generators REQUIRED)

ament_export_dependencies(ament_cmake_core)
ament_export_dependencies(rcpputils)
ament_export_dependencies(rcutils)
ament_export_dependencies(rmw)
ament_export_dependencies(rosidl_default_runtime)

rosidl_generate_interfaces(
  ${PROJECT_NAME}
  "msg/Gid.msg"
  "msg/NodeEntitiesInfo.msg"
  "msg/ParticipantEntitiesInfo.msg"
)

add_library(${PROJECT_NAME}_library SHARED
  src/gid_utils.cpp
  src/graph_cache.cpp)
set_target_properties(${PROJECT_NAME}_library
  PROPERTIES OUTPUT_NAME ${PROJECT_NAME})
ament_target_dependencies(${PROJECT_NAME}_library
  "rcpputils"
  "rcutils"
  "rmw"
  "rosidl_runtime_cpp")
add_dependencies(${PROJECT_NAME}_library
  ${PROJECT_NAME})
target_include_directories(${PROJECT_NAME}_library PUBLIC
  include
  ${CMAKE_CURRENT_BINARY_DIR}/rosidl_generator_c
  ${CMAKE_CURRENT_BINARY_DIR}/rosidl_generator_cpp)

# Causes the visibility macros to use dllexport rather than dllimport,
# which is appropriate when building the dll but not consuming it.
target_compile_definitions(${PROJECT_NAME}_library
  PRIVATE "RMW_DDS_COMMON_BUILDING_LIBRARY")

install(
  TARGETS ${PROJECT_NAME}_library
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

ament_export_include_directories(include)
ament_export_libraries(${PROJECT_NAME}_library)

install(
  DIRECTORY include/
  DESTINATION include)

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  find_package(ament_cmake_gmock REQUIRED)
  ament_lint_auto_find_test_dependencies()

  ament_add_gmock(test_graph_cache test/test_graph_cache.cpp)
  if(TARGET test_graph_cache)
    target_link_libraries(test_graph_cache ${PROJECT_NAME}_library)
  endif()
endif()

ament_package()
