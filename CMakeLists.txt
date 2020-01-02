cmake_minimum_required(VERSION 3.7)
project(Amnesia VERSION 0.1.0 LANGUAGES CXX)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

#set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake)
set(CMAKE_MODULE_PATH ${CMAKE_SOURCE_DIR}/cmake ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}/cmake")


option (FORCE_COLORED_OUTPUT "Always produce ANSI-colored output (GNU/Clang only)." TRUE)
if (${FORCE_COLORED_OUTPUT})
    if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
       add_compile_options (-fdiagnostics-color=always)
    elseif ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
       add_compile_options (-fcolor-diagnostics)
    endif ()
endif ()

set(GLFW_BUILD_DOCS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_TESTS OFF CACHE BOOL "" FORCE)
set(GLFW_BUILD_EXAMPLES OFF CACHE BOOL "" FORCE)

##############################################
# Declare dependencies
#find_package(Boost REQUIRED)
find_package(Vulkan REQUIRED)
find_package(GLFW REQUIRED)

##############################################
# Create target and set properties

add_library(Amnesia SHARED
  src/core.cpp
)

#Add an alias so that library can be used inside the build tree, e.g. when testing
add_library(Amnesia::Amnesia ALIAS Amnesia)

#Set target properties
target_include_directories(Amnesia
  PUBLIC
    $<INSTALL_INTERFACE:include>
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
  PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/src
    Vulkan::Vulkan
    ${GLFW_INCLUDE_DIR}
)

target_link_libraries(Amnesia
  PUBLIC
  PRIVATE
    Vulkan::Vulkan
    ${GLFW_LIBRARY}
)

##############################################
# Installation instructions

include(GNUInstallDirs)
set(INSTALL_CONFIGDIR ${CMAKE_INSTALL_LIBDIR}/cmake/Amnesia)

install(TARGETS Amnesia
    EXPORT Amnesia-targets
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

#This is required so that the exported target has the name Amnesia and not amnesia
set_target_properties(Amnesia PROPERTIES
  EXPORT_NAME Core
)

install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})

#Export the targets to a script
install(EXPORT Amnesia-targets
  FILE
  AmnesiaTargets.cmake
  NAMESPACE
  Amnesia::
  DESTINATION
    ${INSTALL_CONFIGDIR}
)

#Create a ConfigVersion.cmake file
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    ${CMAKE_CURRENT_BINARY_DIR}/AmnesiaConfigVersion.cmake
    VERSION ${PROJECT_VERSION}
    COMPATIBILITY AnyNewerVersion
)

configure_package_config_file(${CMAKE_CURRENT_LIST_DIR}/cmake/AmnesiaConfig.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/AmnesiaConfig.cmake
    INSTALL_DESTINATION ${INSTALL_CONFIGDIR}
)

#Install the config, configversion and custom find modules
install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/AmnesiaConfig.cmake
    ${CMAKE_CURRENT_BINARY_DIR}/AmnesiaConfigVersion.cmake
    DESTINATION ${INSTALL_CONFIGDIR}
)

##############################################
## Exporting from the build tree

export(EXPORT Amnesia-targets FILE ${CMAKE_CURRENT_BINARY_DIR}/AmnesiaTargets.cmake NAMESPACE Amnesia::)

#Register package in user's package registry
#export(PACKAGE Amnesia)


##############################################
add_executable(amnesiaInfo
  src/app.cpp
)

target_link_libraries(amnesiaInfo
  PUBLIC
  PRIVATE
    Amnesia::Amnesia
    Vulkan::Vulkan
)
