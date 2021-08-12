cmake_minimum_required(VERSION 3.8)

project(DynCam)

### Helper functions
list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake")
include(DynCamHelpers)

# we want to link our static lib into a shared. It is not sufficient to just mark the main lib
# with pic. So we do it once and for all.
set(CMAKE_POSITION_INDEPENDENT_CODE ON)

if (CMAKE_CXX_COMPILER_ID STREQUAL "GNU")
    if (CMAKE_CXX_COMPILER_VERSION VERSION_LESS "6.1")
        message(FATAL_ERROR "Insufficient gcc version")
    endif()
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    if (CMAKE_CXX_COMPILER_VERSION VERSION_LESS "15")
        message(FATAL_ERROR "Insufficient msvc version")
    endif()
elseif (CMAKE_CXX_COMPILER_ID STREQUAL "Clang")
	if (CMAKE_CXX_COMPILER_VERSION VERSION_LESS "3.4")
        message(FATAL_ERROR "Insufficient clang version")
    endif()
endif()

if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
	### compiler settings
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wpedantic")
	# cpp.react uses old boost coroutines. we want to ignore this warning
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -DBOOST_COROUTINES_NO_DEPRECATION_WARNING")
	# library warings we don't care about
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wno-gnu -Wno-dangling-field -Wno-mismatched-tags")
endif()

### Required modules that are not installed on the system
list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/modules")

### msvc multi core compile
if (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /MP")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /bigobj")
endif()

### compile with static libs on windows for unreal compatibility
if (CMAKE_CXX_COMPILER_ID STREQUAL "MSVC")
    set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} /MT")
    set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} /MTd")
endif()

### benchmarking and logging
option(enable_logging "Log timings and graphs" OFF)
if (enable_logging)
    add_definitions(-DREACT_ENABLE_LOGGING)
endif()

cmake_policy(SET CMP0074 NEW)

### dependencies
find_package(tbb REQUIRED)
list(APPEND DYNCAM_COMMON_INCLUDE_DIRS "${TBB_INCLUDE_DIRS}")
list(APPEND DYNCAM_COMMON_LIBRARIES "${TBB_LIBRARIES}")

add_subdirectory(deps/googletest)
set(INSTALL_GTEST FALSE CACHE BOOL "Enable installation of googletest. (Projects embedding googletest may want to turn this OFF.)" FORCE) # don't install gtest globally

add_subdirectory(deps/cpp.react)
list(APPEND DYNCAM_COMMON_INCLUDE_DIRS ${PROJECT_SOURCE_DIR}/deps/cpp.react/include)
list(APPEND DYNCAM_COMMON_LIBRARIES CppReact)

find_package(OpenCV REQUIRED)
list(APPEND DYNCAM_COMMON_INCLUDE_DIRS ${OpenCV_INCLUDE_DIRS})
list(APPEND DYNCAM_COMMON_LIBRARIES ${OpenCV_LIBS})
# opencv does not know RelWithDebInfo. Map it to RELEASE
set_target_properties(${OpenCV_LIBS} PROPERTIES MAP_IMPORTED_CONFIG_RELWITHDEBINFO RELEASE)

find_package(cereal REQUIRED) # header only
list(APPEND DYNCAM_COMMON_INCLUDE_DIRS "${CEREAL_INCLUDE_DIRS}")

find_package(lz4 REQUIRED)
if (LZ4_FOUND)
    list(APPEND DYNCAM_COMMON_INCLUDE_DIRS ${LZ4_INCLUDE_DIRS})
    list(APPEND DYNCAM_COMMON_LIBRARIES ${LZ4_LIBRARIES})
endif()


# includes and libs for all projects
list(APPEND DYNCAM_COMMON_INCLUDE_DIRS ${PROJECT_SOURCE_DIR}/src)


### assemble our targets
add_subdirectory(src)
