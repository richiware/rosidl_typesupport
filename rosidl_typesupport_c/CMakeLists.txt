cmake_minimum_required(VERSION 3.5)

project(rosidl_typesupport_c)

if(NOT WIN32)
  set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -std=c11 -Wall -Wextra")
  set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++14 -Wall -Wextra")
endif()

find_package(ament_cmake REQUIRED)
# provides FindPoco.cmake and Poco on platforms without it
find_package(poco_vendor)
find_package(Poco COMPONENTS Foundation)
find_package(rosidl_generator_c REQUIRED)

link_directories(${Poco_LIBRARY_DIR})

ament_export_dependencies(rosidl_typesupport_interface)

ament_export_include_directories(include)

ament_python_install_package(${PROJECT_NAME})

add_library(${PROJECT_NAME} SHARED
  src/identifier.c
  src/message_type_support_dispatch.cpp
  src/service_type_support_dispatch.cpp
  src/type_support_dispatch.cpp)
if(Poco_FOUND)
  target_compile_definitions(${PROJECT_NAME}
    PRIVATE "ROSIDL_TYPESUPPORT_C_USE_POCO")
endif()
if(WIN32)
  target_compile_definitions(${PROJECT_NAME}
    PRIVATE "ROSIDL_TYPESUPPORT_C_BUILDING_DLL")
endif()
target_include_directories(${PROJECT_NAME}
  PUBLIC
  include ${Poco_INCLUDE_DIRS}
)
target_link_libraries(${PROJECT_NAME} ${Poco_LIBRARIES})
ament_target_dependencies(${PROJECT_NAME} "rosidl_generator_c")
ament_export_libraries(${PROJECT_NAME})

if(BUILD_TESTING)
  find_package(ament_lint_auto REQUIRED)
  ament_lint_auto_find_test_dependencies()
endif()

ament_package(
  CONFIG_EXTRAS "rosidl_typesupport_c-extras.cmake.in"
)

install(
  PROGRAMS bin/rosidl_typesupport_c
  DESTINATION lib/rosidl_typesupport_c
)
install(
  DIRECTORY cmake resource
  DESTINATION share/${PROJECT_NAME}
)
install(
  DIRECTORY include/
  DESTINATION include
)
install(
  TARGETS ${PROJECT_NAME}
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)