cmake_minimum_required(VERSION 3.8 FATAL_ERROR)

project("ViennaLS" VERSION 2.0.0)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

add_definitions(-DVIENNALS_VERSION=${PROJECT_VERSION})

# c++17 for inlince constexpr variables
set(CMAKE_CXX_STANDARD "17")

# set default build type
set(DEFAULT_BUILD_TYPE "Release")
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
  message(STATUS "Setting build type to '${DEFAULT_BUILD_TYPE}' as none was specified.")
  set(CMAKE_BUILD_TYPE
      "${DEFAULT_BUILD_TYPE}"
      CACHE STRING "Choose the type of build." FORCE)
  # Set the possible values of build type for cmake-gui
  set_property(CACHE CMAKE_BUILD_TYPE PROPERTY STRINGS "Debug" "Release" "MinSizeRel"
                                               "RelWithDebInfo")
endif()

# tell VS to export all symbols to its dll files
if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "MSVC")
  set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS
      TRUE
      CACHE BOOL "Export all symbols" FORCE)
endif()

option(VIENNALS_STATIC_BUILD "Link dependencies statically." OFF)
option(VIENNALS_BUILD_SHARED_LIBS "Build template specialisations for shorter compile times." OFF)
option(VIENNALS_BUILD_TESTS "Build tests." OFF)
option(VIENNALS_BUILD_EXAMPLES "Build examples." OFF)

# whether to use VTK or not
option(VIENNALS_USE_VTK "Build with VTK file support." ON)
if(VIENNALS_USE_VTK)
  list(APPEND VIENNALS_DEPENDENCIES "vtk_external")
endif()

option(VIENNALS_BUILD_PYTHON_2 "Build for python2.x." OFF)
option(VIENNALS_BUILD_PYTHON_3 "Build for python3.x. Trumps python2.x build." OFF)
if(VIENNALS_BUILD_PYTHON_2 OR VIENNALS_BUILD_PYTHON_3)
  list(APPEND VIENNALS_DEPENDENCIES "pybind11_external")
endif()

# build dependencies
set(DEPENDENCIES_DIR ${CMAKE_SOURCE_DIR}/dependencies)
# external project base directory for all projects in this hierarchy
set_property(DIRECTORY PROPERTY EP_BASE ${DEPENDENCIES_DIR})

# Include all external dependencies
include(ExternalProject)
add_subdirectory(external/upstream)

list(APPEND VIENNALS_DEPENDENCIES "viennahrle_external")

include(ExternalProject)

ExternalProject_Add(
  ${PROJECT_NAME}_core
  DEPENDS ${VIENNALS_DEPENDENCIES}
  SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src
  CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
             -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
             -DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}
             -DCMAKE_CXX_STANDARD=${CMAKE_CXX_STANDARD}
             -DCMAKE_CXX_EXTENSIONS=${CMAKE_CXX_EXTENSIONS}
             -DCMAKE_CXX_STANDARD_REQUIRED=${CMAKE_CXX_STANDARD_REQUIRED}
             -DCMAKE_PROJECT_VERSION=${PROJECT_VERSION}
             # Paths to dependencies
             -DViennaHRLE_DIR=${ViennaHRLE_DIR}
             -DVTK_DIR=${VTK_DIR}
             -Dpybind11_DIR=${pybind11_DIR}
             # Configuration Options
             -DVIENNALS_USE_VTK=${VIENNALS_USE_VTK}
             -DVIENNALS_BUILD_EXAMPLES=${VIENNALS_BUILD_EXAMPLES}
             -DVIENNALS_BUILD_TESTS=${VIENNALS_BUILD_TESTS}
             -DVIENNALS_BUILD_SHARED_LIBS=${VIENNALS_BUILD_SHARED_LIBS}
             -DVIENNALS_BUILD_PYTHON_2=${VIENNALS_BUILD_PYTHON_2}
             -DVIENNALS_BUILD_PYTHON_3=${VIENNALS_BUILD_PYTHON_3}
             -DVIENNALS_STATIC_BUILD=${VIENNALS_STATIC_BUILD}
  CMAKE_CACHE_ARGS -DCMAKE_CXX_FLAGS:STRING=${CMAKE_CXX_FLAGS}
                   -DCMAKE_PREFIX_PATH:PATH=${CMAKE_PREFIX_PATH}
  BUILD_ALWAYS 1
  INSTALL_COMMAND "")
