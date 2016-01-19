cmake_minimum_required(VERSION 2.8.3)
project(pangolin_catkin)

find_package(catkin_simple REQUIRED)
catkin_simple(ALL_DEPS_REQUIRED)

include(ExternalProject)

option(PANGOLIN_USE_GUI "Build 3rdparty pangolin package with GUI support" ON)
option(PANGOLIN_USE_VIDEO "Build 3rdparty pangolin package with Video support" OFF)
option(PANGOLIN_USE_OCULUS "Build 3rdparty pangolin package with Oculus support" OFF)
option(PANGOLIN_USE_VARS "Build 3rdparty pangolin package with Vars support" OFF)

find_package(FREEGLUT QUIET)
find_package(GLUT QUIET)

set(PANGOLIN_CMAKE_OPTIONS "")
if (PANGOLIN_USE_GUI)
  list(APPEND PANGOLIN_CMAKE_OPTIONS "-DBUILD_PANGOLIN_GUI:BOOL=ON")
  find_package(OpenGL REQUIRED QUIET)
  find_package(GLEW REQUIRED QUIET)
else()
  list(APPEND PANGOLIN_CMAKE_OPTIONS "-DBUILD_PANGOLIN_GUI:BOOL=OFF")
endif()

if (PANGOLIN_USE_VIDEO)
  list(APPEND PANGOLIN_CMAKE_OPTIONS "-DBUILD_PANGOLIN_VIDEO:BOOL=ON")
  find_package(DepthSense)
else()
  list(APPEND PANGOLIN_CMAKE_OPTIONS "-DBUILD_PANGOLIN_VIDEO:BOOL=OFF")
endif()

if (PANGOLIN_USE_OCULUS)
  list(APPEND PANGOLIN_CMAKE_OPTIONS "-DBUILD_PANGOLIN_OCULUS:BOOL=ON")
else()
  list(APPEND PANGOLIN_CMAKE_OPTIONS "-DBUILD_PANGOLIN_OCULUS:BOOL=OFF")
endif()

if (PANGOLIN_USE_VARS)
  list(APPEND PANGOLIN_CMAKE_OPTIONS "-DBUILD_PANGOLIN_VARS:BOOL=ON")
else()
  list(APPEND PANGOLIN_CMAKE_OPTIONS "-DBUILD_PANGOLIN_VARS:BOOL=OFF")
endif()

if (CMAKE_BUILD_TYPE)
  list(APPEND PANGOLIN_CMAKE_OPTIONS "-DCMAKE_BUILD_TYPE:String=${CMAKE_BUILD_TYPE}")
endif()

file(MAKE_DIRECTORY ${CATKIN_DEVEL_PREFIX}/include)
ExternalProject_Add(pangolin_EXTERNAL
  GIT_REPOSITORY https://github.com/stevenlovegrove/Pangolin
  UPDATE_COMMAND ""
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=${CATKIN_DEVEL_PREFIX}
    -DBUILD_SHARED_LIBS:BOOL=ON
    ${PANGOLIN_CMAKE_OPTIONS}
    -DBUILD_EXAMPLES:BOOL=OFF
    -DCPP11_NO_BOOST:BOOL=ON
    BUILD_COMMAND ${CMAKE_COMMAND} --build ./ --target install
    INSTALL_COMMAND ""
   )

ExternalProject_CatkinInstall(pangolin_EXTERNAL)

cs_add_library(${PROJECT_NAME} src/empty.cpp)
add_dependencies(${PROJECT_NAME} pangolin_EXTERNAL)

if (EXISTS "${CATKIN_DEVEL_PREFIX}/lib/libpangolin${CMAKE_SHARED_LIBRARY_SUFFIX}")
  SET(PANGOLIN_FOUND TRUE)
else ()
  SET(PANGOLIN_FOUND FALSE)
endif ()

cs_install()
cs_export(INCLUDE_DIRS ${CATKIN_DEVEL_PREFIX}/include
  CFG_EXTRAS pangolin-extras.cmake
  )
