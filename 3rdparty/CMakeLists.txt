################################################################################
# Generate/update/build all 3rdparty and dependency projects
################################################################################

function(updateSubmodule workingRelativeDirectory submodulePath)
execute_process(COMMAND ${GIT_EXECUTABLE} submodule update --init ${submodulePath}
                    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/${workingRelativeDirectory}
                    RESULT_VARIABLE GIT_SUBMOD_RESULT)
					
if(NOT GIT_SUBMOD_RESULT EQUAL "0")
	message(FATAL_ERROR "git submodule update --init failed with ${GIT_SUBMOD_RESULT}, please checkout submodules")
endif()
endfunction()

#execute_process(COMMAND ${GIT_EXECUTABLE} submodule update --init
#                    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
#                    RESULT_VARIABLE GIT_SUBMOD_RESULT)

################################################################################
# Global configuration types
################################################################################
set(CMAKE_CONFIGURATION_TYPES
    "Debug"
    "Release"
    CACHE STRING "" FORCE
)

################################################################################
# Global compiler options
################################################################################
if(MSVC)
    set(CMAKE_CXX_FLAGS "")
    set(CMAKE_CXX_FLAGS_DEBUG "/MTd /Zi /Ob0 /Od /RTC1" CACHE STRING "" FORCE)
    set(CMAKE_CXX_FLAGS_RELEASE "/MT /O2 /Ob2 /DNDEBUG" CACHE STRING "" FORCE)
endif()

################################################################################
# Global linker options
################################################################################
if(MSVC)
    # remove default flags provided with CMake for MSVC
    set(CMAKE_EXE_LINKER_FLAGS "")
    set(CMAKE_MODULE_LINKER_FLAGS "")
    set(CMAKE_SHARED_LINKER_FLAGS "")
    set(CMAKE_STATIC_LINKER_FLAGS "")
    set(CMAKE_EXE_LINKER_FLAGS_DEBUG "${CMAKE_EXE_LINKER_FLAGS}")
    set(CMAKE_MODULE_LINKER_FLAGS_DEBUG "${CMAKE_MODULE_LINKER_FLAGS}")
    set(CMAKE_SHARED_LINKER_FLAGS_DEBUG "${CMAKE_SHARED_LINKER_FLAGS}")
    set(CMAKE_STATIC_LINKER_FLAGS_DEBUG "${CMAKE_STATIC_LINKER_FLAGS}")
    set(CMAKE_EXE_LINKER_FLAGS_RELEASE "${CMAKE_EXE_LINKER_FLAGS}")
    set(CMAKE_MODULE_LINKER_FLAGS_RELEASE "${CMAKE_MODULE_LINKER_FLAGS}")
    set(CMAKE_SHARED_LINKER_FLAGS_RELEASE "${CMAKE_SHARED_LINKER_FLAGS}")
    set(CMAKE_STATIC_LINKER_FLAGS_RELEASE "${CMAKE_STATIC_LINKER_FLAGS}")
endif()

################################################################################
# Project dependencies
################################################################################

set(BOOST_ROOT ${VPD_ROOT_PATH}/3rdparty/boost/boost)
set(Boost_DIR ${BOOST_ROOT}/stage/lib/cmake/Boost-1.74.0 CACHE FILEPATH "" FORCE)
set(Boost_NO_SYSTEM_PATHS ON)

add_custom_target(boost
		COMMAND cmd /C bootstrap.bat --prefix=BAWGeneratedBuild --with-toolset=msvc --with-libraries=date_time,filesystem,regex,system,program_options,thread,iostreams,atomic
		COMMAND cmd /C b2.exe link=static --with-date_time --with-filesystem --with-regex --with-system --with-program_options --with-thread --with-iostreams --with-atomic
		WORKING_DIRECTORY ${BOOST_ROOT}/
		COMMENT "Building Boost..."
		VERBATIM
	)
	
list(APPEND LAS_CMAKE_ARGS "-DBOOST_ROOT:PATH=${BOOST_ROOT}")
list(APPEND LAS_CMAKE_ARGS "-DBoost_DIR:PATH=${Boost_DIR}")
list(APPEND LAS_CMAKE_ARGS "-DBUILD_OSGEO4W:BOOL=FALSE")
list(APPEND LAS_CMAKE_ARGS "-DWITH_GEOTIFF:BOOL=FALSE")
list(APPEND LAS_CMAKE_ARGS "-DWITH_LASZIP:BOOL=TRUE")
list(APPEND LAS_CMAKE_ARGS "-DWITH_STATIC_LASZIP:BOOL=TRUE")
list(APPEND LAS_CMAKE_ARGS "-DLASZIP_INCLUDE_DIR:PATH=${VPD_ROOT_PATH}/3rdparty/liblaszip/liblaszip/include")
list(APPEND LAS_CMAKE_ARGS "-DLASZIP_LIBRARY:FILEPATH=${VPD_ROOT_PATH}/3rdparty/liblaszip/liblaszip/build/bin/$(Configuration)/laszip.lib")

ExternalProject_Add(liblas
	DOWNLOAD_COMMAND  ""
	SOURCE_DIR        "${VPD_ROOT_PATH}/3rdparty/liblas/liblas"
	BINARY_DIR        "${VPD_ROOT_PATH}/3rdparty/liblas/liblas/build"
	CMAKE_ARGS		  ${LAS_CMAKE_ARGS}
	INSTALL_COMMAND   ""
	TEST_COMMAND      ""
)

ExternalProject_Add(liblaszip
	DOWNLOAD_COMMAND  ""
	SOURCE_DIR        "${VPD_ROOT_PATH}/3rdparty/liblaszip/liblaszip"
	BINARY_DIR        "${VPD_ROOT_PATH}/3rdparty/liblaszip/liblaszip/build"
	INSTALL_COMMAND   ""
	TEST_COMMAND      ""
)

add_dependencies(liblas liblaszip boost)

ExternalProject_Add(eigen3
	DOWNLOAD_COMMAND  ""
	SOURCE_DIR        "${VPD_ROOT_PATH}/3rdparty/eigen/eigen"
	BINARY_DIR        "${VPD_ROOT_PATH}/3rdparty/eigen/eigen/build"
	INSTALL_COMMAND   cmake --install "${VPD_ROOT_PATH}/3rdparty/eigen/eigen/build" --prefix "${VPD_ROOT_PATH}/3rdparty/eigen/eigen/build/install"
	TEST_COMMAND      ""
	USES_TERMINAL_INSTALL 1
)

ExternalProject_Add(flann
	DOWNLOAD_COMMAND  ""
	SOURCE_DIR        "${VPD_ROOT_PATH}/3rdparty/flann/flann"
	BINARY_DIR        "${VPD_ROOT_PATH}/3rdparty/flann/flann/build"
	INSTALL_COMMAND   cmake --install "${VPD_ROOT_PATH}/3rdparty/flann/flann/build" --prefix "${VPD_ROOT_PATH}/3rdparty/flann/flann/install"
	TEST_COMMAND      ""
	USES_TERMINAL_INSTALL 1
)

# patched setup is placed in the PCL's cmake since it is a fork (contains options, boost, flann and eigen dirs)
add_subdirectory(${VPD_ROOT_PATH}/3rdparty/pcl/pcl ${VPD_ROOT_PATH}/3rdparty/pcl/pcl)

set(3RDPARTY_PCL_TARGETS
	pcl_common												
	pcl_filters										
	pcl_kdtree									
	pcl_surface			
	pcl_search
)

foreach(trgt IN LISTS 3RDPARTY_PCL_TARGETS)
		set_property(TARGET ${trgt} PROPERTY
             MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")
endforeach()

add_custom_target(PCL)
add_custom_target(PCL_INSTALL
	COMMAND cmake --install "build/" --prefix "build/install/" || EXIT /B 0 # since there are still issues with static compiling on latest tags of PCL, I have to override exit status
	WORKING_DIRECTORY ${VPD_ROOT_PATH}/3rdparty/pcl/pcl
	USES_TERMINAL
	VERBATIM
)

add_dependencies(PCL
	eigen3
	flann
	pcl_filters										
	pcl_kdtree									
	pcl_surface			
	pcl_search
	PCL_INSTALL
)