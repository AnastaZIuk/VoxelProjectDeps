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

set(_CONFIG_BUILD_FILE_ ${VPD_ROOT_PATH}/config/BuildConfigOptions.h)
set(_DLL_LIBLAS_DIR_ ${VPD_ROOT_PATH}/3rdparty/liblas/liblas/build/bin)
set(_DLL_LIBLASZIP_DIR_ ${VPD_ROOT_PATH}/3rdparty/liblaszip/liblaszip/build/bin)
set(_DLL_PCL_DIR_ ${VPD_ROOT_PATH}/3rdparty/pcl/pcl/build/install/bin)

set(_PCL_LIBLAS_DLL_DIRS_DYNAMIC_LINKING_
	"${_DLL_LIBLAS_DIR_}/Release/liblas.dll"
	"${_DLL_LIBLAS_DIR_}/Debug/liblasd.dll"
	"${_DLL_LIBLASZIP_DIR_}/Release/laszip.dll"
	"${_DLL_LIBLASZIP_DIR_}/Debug/laszipd.dll"
	"${_DLL_PCL_DIR_}/pcl_common.dll"
	"${_DLL_PCL_DIR_}/pcl_kdtree.dll"
	"${_DLL_PCL_DIR_}/pcl_octree.dll"
	"${_DLL_PCL_DIR_}/pcl_surface.dll"
	"${_DLL_PCL_DIR_}/pcl_search.dll"
)

get_filename_component(_DLL_INSTALL_ "../../../super_secret_cpu-gpu_reseach/PointCloud2DistanceField/bin" ABSOLUTE)
install(FILES ${_PCL_LIBLAS_DLL_DIRS_DYNAMIC_LINKING_} DESTINATION ${_DLL_INSTALL_} OPTIONAL)