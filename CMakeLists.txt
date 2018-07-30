IF (WIN32)
  CMAKE_MINIMUM_REQUIRED(VERSION 3.11)
ELSE()
  CMAKE_MINIMUM_REQUIRED(VERSION 3.2)
ENDIF()

# Version
set(OPENEXR_VERSION_MAJOR 2)
set(OPENEXR_VERSION_MINOR 3)
set(OPENEXR_VERSION_PATCH 0)
set(OPENEXR_VERSION ${OPENEXR_VERSION_MAJOR}.${OPENEXR_VERSION_MINOR}.${OPENEXR_VERSION_PATCH})
set(OPENEXR_VERSION_API ${OPENEXR_VERSION_MAJOR}_${OPENEXR_VERSION_MINOR})

project(OpenEXR VERSION ${OPENEXR_VERSION})

#-------------------------------------------------------------------------------
# The following user options are cached. They are named with the OPENEXR
# prefix in order that they be grouped together in tools such as ccmake and cmake-gui.

option(OPENEXR_BUILD_ILMBASE        "Build IlmBase"             ON)
option(OPENEXR_BUILD_OPENEXR        "Build OpenEXR"             ON)
option(OPENEXR_BUILD_PYTHON_LIBS    "Build the Python bindings" OFF)
option(OPENEXR_BUILD_VIEWERS        "Build the viewers"         OFF)
option(OPENEXR_BUILD_TESTS          "Enable the tests"          ON)

option(OPENEXR_BUILD_SHARED    "Build Shared Libraries"    ON)
option(OPENEXR_BUILD_STATIC    "Build Static Libraries"    OFF)
option(OPENEXR_NAMESPACE_VERSIONING "Use Namespace Versioning"  ON)
option(OPENEXR_FORCE_CXX03          "Force CXX03"               OFF)
set(OPENEXR_PYTHON_MAJOR "2" CACHE STRING "Python major version")
set(OPENEXR_PYTHON_MINOR "7" CACHE STRING "Python minor version")

# For more info on finding boost python:
# https://cmake.org/cmake/help/v3.11/module/FindBoost.html

# end of user options
#-------------------------------------------------------------------------------


if (OPENEXR_BUILD_VIEWERS AND NOT OPENEXR_BUILD_OPENEXR)
  message(ERROR, "Configuration error, enable OPENEXR_BUILD_OPENEXR for OPENEXR_BUILD_VIEWERS")
endif()

if (WIN32 AND OPENEXR_BUILD_ILMBASE AND OPENEXR_BUILD_OPENEXR AND OPENEXR_BUILD_SHARED)
    # necessary for building dwa lookup tables, and b44log tables on windows
    set(BUILD_ILMBASE_STATIC ON)
elseif (OPENEXR_BUILD_ILMBASE AND OPENEXR_BUILD_STATIC)
    set(BUILD_ILMBASE_STATIC ON)
else()
    set(BUILD_ILMBASE_STATIC OFF)
endif()

# Testing
set(ENABLE_TESTS ${OPENEXR_BUILD_TESTS})
if(ENABLE_TESTS)
  include(CTest)
  enable_testing()
endif()

# CPACK
set(CPACK_PROJECT_NAME             ${PROJECT_NAME})
set(CPACK_PROJECT_VERSION          ${PROJECT_VERSION})
set(CPACK_SOURCE_IGNORE_FILES      "/.git*;/.cvs*;${CPACK_SOURCE_IGNORE_FILES}")
set(CPACK_SOURCE_GENERATOR         "TGZ")
set(CPACK_SOURCE_PACKAGE_FILE_NAME "${CMAKE_PROJECT_NAME}-${OPENEXR_VERSION}" )
include(CPack)

# Configuration
if(OPENEXR_FORCE_CXX03)
  ADD_DEFINITIONS ( -std=c++03 )
else(OPENEXR_FORCE_CXX03)
  # VP18 switches to c++14, so let's do that by default
  set(CMAKE_CXX_STANDARD 14 CACHE STRING "C++ ISO Standard")
  # but switch gnu++14 or other extensions off for portability
  set(CMAKE_CXX_EXTENSIONS OFF)
endif()

add_definitions( -DHAVE_CONFIG_H )

if(NOT WIN32)
  add_definitions( -pthread )
endif()

if(WIN32)
  set(RUNTIME_DIR bin)
else()
  set(RUNTIME_DIR lib)
endif()

set(OPENEXR_LIBSUFFIX "")
set(ILMBASE_LIBSUFFIX "")
if(OPENEXR_NAMESPACE_VERSIONING)
  set( OPENEXR_LIBSUFFIX "-${OPENEXR_VERSION_API}" )
  set( ILMBASE_LIBSUFFIX "-${OPENEXR_VERSION_API}" )
endif()

# MacOs/linux rpathing
set(CMAKE_MACOSX_RPATH 1)
set(BUILD_WITH_INSTALL_RPATH 1)

# Packages
find_package(ZLIB)
if(OPENEXR_BUILD_OPENEXR AND NOT ZLIB_FOUND)
  message(WARNING "Make Zlib available to CMake's search path, and re-run configuration")
  message(FATAL_ERROR "Zlib is a required dependency when OPENEXR_BUILD_OPENEXR is set")
endif()

if (OPENEXR_BUILD_PYTHON_LIBS)
  find_package(PythonLibs ${OPENEXR_PYTHON_MAJOR}.${OPENEXR_PYTHON_MINOR})
  if(NOT PythonLibs_FOUND)
    message(WARNING "Make Python ${OPENEXR_PYTHON_MAJOR}.${OPENEXR_PYTHON_MINOR} available to CMake's search path, and re-run configuration")
    message(FATAL_ERROR "Python is a required dependency when OPENEXR_BUILD_PYTHON_LIBS is set")
  endif()

  find_package(Boost COMPONENTS python${OPENEXR_PYTHON_MAJOR}${OPENEXR_PYTHON_MINOR})
  if(NOT Boost_PYTHON${OPENEXR_PYTHON_MAJOR}${OPENEXR_PYTHON_MINOR}_FOUND)
    message(WARNING "Make boost Python${OPENEXR_PYTHON_MAJOR}${OPENEXR_PYTHON_MINOR} available to CMake's search path, and re-run configuration")
    message(FATAL_ERROR "boost Python is a required dependency when OPENEXR_BUILD_PYTHON_LIBS is set")
  endif()

  find_package(NumPy)
  if(NOT NumPy_FOUND)
    message(WARNING "Numpy not found, PyImathNumpy will not be built")
  endif()
endif()

if(OPENEXR_BUILD_VIEWERS)
  message(WARNING "Viewers are currently out of order. Building anyway")
  SET (FLTK_SKIP_FLUID 1)
  find_package(FLTK)
  if(NOT FLTK_FOUND)
    message(WARNING "FLTK not found, exrdisplay will not be built")
  endif()
  find_package(OpenGL)
  if(NOT OpenGL_FOUND)
    message(WARNING "OpenGL not found, exrdisplay will not be built")
  endif()
endif()

# Perform the build
if(OPENEXR_BUILD_ILMBASE)
    add_subdirectory(IlmBase)
endif()
if(OPENEXR_BUILD_OPENEXR)
    add_subdirectory(OpenEXR)
else()
    message(INFO, "Skipping OpenEXR build")
endif()
if(OPENEXR_BUILD_PYTHON_LIBS)
    add_subdirectory(PyIlmBase)
else()
    message(INFO, "Skipping Python bindings")
endif()
if(OPENEXR_BUILD_VIEWERS AND FLTK_FOUND AND OpenGL_FOUND)
    add_subdirectory(OpenEXR_Viewers)
else()
    message(INFO, "Skipping OpenEXR Viewers")
endif()