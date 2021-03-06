# SPDX-License-Identifier: BSD-3-Clause
# Copyright Contributors to the OpenEXR Project.

cmake_minimum_required(VERSION 3.12)

if(POLICY CMP0074)
  cmake_policy(SET CMP0074 NEW)
endif()

if(POLICY CMP0077)
  # enable variables set outside to override options
  cmake_policy(SET CMP0077 NEW)
endif()

# Version numbers are maintained in config/version.cmake
include(config/version.cmake)

project(Imath VERSION ${IMATH_VERSION} LANGUAGES C CXX)

# ImathSetup.cmake declares all the configuration variables visible
# in cmake-gui or similar and the rest of the global
# project setup. Check the context to see what is configurable.
include(config/ImathSetup.cmake)

# Config headers and package config files
add_subdirectory(config)

# Utility function for the repeated boilerplate of defining the libraries
include(config/LibraryDefine.cmake)

# Source code is in src/Imath
add_subdirectory(src/Imath)

# Imath_DIR points to the location of ImathConfig.cmake, which tells
# downstream projects where to find Imath, via find_package(Imath).
set(Imath_DIR "${CMAKE_CURRENT_BINARY_DIR}/config" CACHE PATH "" FORCE)

# Add an empty ImathTargets.cmake file for the config to use. It can
# be empty since we already defined the targets in add_subdirectory().
file(WRITE "${CMAKE_CURRENT_BINARY_DIR}/config/ImathTargets.cmake" "# Dummy file")

option(PYTHON "Set ON to compile PyImath bindings")
if (PYTHON)
  add_subdirectory(src/python)
endif()

# If you want to use ctest to configure, build and
# upload the results, cmake has builtin support for
# submitting to CDash, or any server who speaks the
# same protocol
# 
# These settings will need to be set for your environment,
# and then a script such as the example in
#
# cmake/SampleCTestScript.cmake
#
# edited and placed into the CI system, then run:
#
# cmake -S cmake/SampleCTestScript.cmake
#
# [or whatever you name the file you edit]
# 
#set(CTEST_PROJECT_NAME "Imath")
#set(CTEST_NIGHTLY_START_TIME "01:01:01 UTC")
#set(CTEST_DROP_METHOD "http") # there are others...
#set(CTEST_DROP_SITE "open.cdash.org")
#set(CTEST_DROP_LOCATION "/submit.php?project=MyProject")
#set(CTEST_DROP_SITE_CDASH TRUE)
include(CTest)
if(BUILD_TESTING AND NOT IMATH_IS_SUBPROJECT)
  enable_testing()
  add_subdirectory(src/ImathTest)
endif()

# Including this module will add a `clang-format` target to the build
# if the clang-format executable can be found. Only do this if we are
# top level.
if(NOT IMATH_IS_SUBPROJECT)
  include(cmake/clang-format.cmake)
endif()
