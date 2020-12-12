cmake_minimum_required(VERSION 3.15.0 FATAL_ERROR)

set(CMAKE_SYSTEM_VERSION 10.0 CACHE STRING "" FORCE)

################################################################################
# Global projects - management located at cmake/projects
################################################################################
project(VoxelProjectDeps)

if(NOT ("${CMAKE_VS_PLATFORM_NAME}" STREQUAL "x64"))
	message(FATAL_ERROR "Only x64 build is supported!")
endif()

get_filename_component(VPD_ROOT_PATH "${CMAKE_CURRENT_SOURCE_DIR}" ABSOLUTE)

################################################################################
# Update submodules
################################################################################
function(update_git_submodule _PATH)
	execute_process(COMMAND git submodule update --init --recursive ${_PATH} WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}")
endfunction()

update_git_submodule("3rdparty/")

set(CMAKE_MODULE_PATH ${VPD_ROOT_PATH}/cmake)

################################################################################
# Additional Global Settings(add specific info there)
################################################################################
include(cmake/GlobalSettingsInclude.cmake OPTIONAL)
include(ExternalProject)

find_package(Git QUIET)

if(GIT_FOUND AND EXISTS "${PROJECT_SOURCE_DIR}/.git")
	message(STATUS "Found GIT!")
else()
	message(FATAL_ERROR "There is no GIT found!")
endif()

################################################################################
# Sub-projects
################################################################################

add_subdirectory(3rdparty/)