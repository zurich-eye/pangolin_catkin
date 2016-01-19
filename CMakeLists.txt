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

set(CATKIN_INSTALL_PREFIX ${CMAKE_INSTALL_PREFIX})
find_program (CATKIN_TOOLS_AVAILABLE NAMES catkin)
message(STATUS "CATKIN_TOOLS_AVAILABLE: ${CATKIN_TOOLS_AVAILABLE}")
message(STATUS "CATKIN_INSTALL_PREFIX: ${CATKIN_INSTALL_PREFIX}")

ExternalProject_Get_Property(pangolin_EXTERNAL BINARY_DIR)
message(STATUS "binary_dir: ${BINARY_DIR}")

# check if we install the catkin package
set(CATKIN_INSTALL "False")
if (DEFINED CATKIN_TOOLS_AVAILABLE)
  message(STATUS "catkin is available (${CATKIN_TOOLS_AVAILABLE})")
  exec_program("catkin config  | grep 'Install Packages:' | tr -s ' ' | cut -c19-"
     OUTPUT_VARIABLE CATKIN_INSTALL
     )
else()
  message(AUTHOR_WARNING "catkin tools not installed; add another approach to check for enabled install;")
endif()
message(STATUS "CATKIN_INSTALL: ${CATKIN_INSTALL}")

if (${CATKIN_INSTALL} STREQUAL "True")
  message(STATUS "Package marked for installation.")
  ExternalProject_Add_Step(pangolin_EXTERNAL 
    catkin_install
    DEPENDEES build
    COMMAND ${CMAKE_COMMAND} -DCMAKE_INSTALL_PREFIX:PATH=${CATKIN_INSTALL_PREFIX} ${BINARY_DIR} && ${CMAKE_COMMAND} --build ${BINARY_DIR} --target install
    COMMENT "Installing the external project..."
    )
else ()
  message(STATUS "Package not installed.")
endif()

cs_add_library(${PROJECT_NAME} src/empty.cpp)
add_dependencies(${PROJECT_NAME} pangolin_EXTERNAL)

if(EXISTS "${CATKIN_DEVEL_PREFIX}/lib/libpangolin${CMAKE_SHARED_LIBRARY_SUFFIX}")
  SET(PANGOLIN_FOUND TRUE)
else()
  SET(PANGOLIN_FOUND FALSE)
endif()

cs_install()
cs_export(INCLUDE_DIRS ${CATKIN_DEVEL_PREFIX}/include
  CFG_EXTRAS pangolin-extras.cmake
  )
