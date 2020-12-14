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
    # remove default flags provided with CMake for MSVC
    set(CMAKE_CXX_FLAGS "")
    set(CMAKE_CXX_FLAGS_DEBUG "")
    set(CMAKE_CXX_FLAGS_RELEASE "")
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