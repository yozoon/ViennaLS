cmake_minimum_required(VERSION 3.8 FATAL_ERROR)

project(ViennaLS VERSION 2.0.0)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
# c++17 for inline constexpr variables
set(CMAKE_CXX_STANDARD "17")

add_definitions(-DVIENNALS_VERSION=${PROJECT_VERSION})

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

list(APPEND VIENNALS_DEPENDENCIES "viennahrle_external")

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
add_custom_target(buildDependencies)
add_subdirectory(external/upstream)

# Save the dependencies in the cache
set(VIENNALS_DEPENDENCIES
    ${VIENNALS_DEPENDENCIES}
    CACHE INTERNAL "List of dependencies that need to be built before ViennaLS can be used" FORCE)

include(GNUInstallDirs)

set(ViennaLS_BINARY_DIR
    ${PROJECT_BINARY_DIR}
    CACHE PATH "Path to local ViennaLS installation" FORCE)

find_package(OpenMP REQUIRED)
list(APPEND VIENNALS_LIBRARIES OpenMP::OpenMP_CXX)

if(VIENNALS_USE_VTK)
  add_compile_definitions(VIENNALS_USE_VTK)

  # use VTK_USE_FILE for older versions of VTK
  if(VTK_VERSION VERSION_LESS "9.0")
    message(STATUS "Using old VTK Version: ${VTK_VERSION}")
    set(VTK_COMPONENTS
        "vtkCommonCore vtkIOCore vtkIOXML vtkIOXMLParser vtkFiltersCore vtkFiltersGeneral vtkFiltersGeometry"
    )
    # find_package is only required for populating the VTK_USE_FILE variable. This case requires
    # that an external VTK installation is used, since our externalproject build already provides a
    # newer version of VTK.
    find_package(VTK CONFIG REQUIRED PATHS ${VTK_DIR} COMPONENTS ${VTK_COMPONENTS})
    set(VTK_LIBRARIES
        vtksys;vtkIOCore;vtkexpat;vtklz4;vtkzlib;vtklzma;vtkdoubleconversion;vtkCommonMisc;vtkCommonSystem;vtkIOXML;vtkFiltersCore;vtkFiltersGeneral;vtkFiltersGeometry;vtkCommonExecutionModel;vtkCommonDataModel;vtkCommonColor;vtkIOXMLParser;vtkIOCore;vtkCommonMisc;vtkCommonComputationalGeometry;vtkCommonTransforms;vtkCommonMath;
    )
    include(${VTK_USE_FILE})
  else()
    # Use the new component names instead of the old vtk* ones
    set(VTK_COMPONENTS
        "CommonCore IOCore IOXML IOXMLParser FiltersCore FiltersGeneral FiltersGeometry")
    # For newer versions of VTK, only the target syntax is supported. This also automatically adds
    # the vtk include directories.
    set(VTK_LIBRARIES
        VTK::CommonCore;VTK::IOCore;VTK::IOXML;VTK::IOXMLParser;VTK::FiltersCore;VTK::FiltersGeneral;VTK::FiltersGeometry;
    )
  endif()

  # only link needed vtk libraries for static builds
  if(VIENNALS_STATIC_BUILD)
    message(STATUS "Static build: Linking static VTK libraries")
  endif(VIENNALS_STATIC_BUILD)

  list(APPEND VIENNALS_LIBRARIES ${VTK_LIBRARIES})
  list(APPEND VIENNALS_PYTHON_LIBRARIES ${VTK_LIBRARIES})
endif(VIENNALS_USE_VTK)

# ##################################################################################################
# BUILD SHARED LIBRARIES
# ##################################################################################################
option(VIENNALS_BUILD_SHARED_LIBS "Build template specialisations for shorter compile times." OFF)
if(VIENNALS_BUILD_SHARED_LIBS)
  list(APPEND VIENNALS_LIBRARIES ViennaHRLE)
  # find_package(ViennaHRLE REQUIRED PATHS ${ViennaHRLE_DIR})

  # add all specialisation source files
  file(GLOB SPECIALISATION_CPPS "lib/*.cpp")

  # build static or shared object for library
  if(VIENNALS_STATIC_BUILD)
    add_library(${PROJECT_NAME} STATIC ${SPECIALISATION_CPPS})
  else(VIENNALS_STATIC_BUILD)
    add_library(${PROJECT_NAME} SHARED ${SPECIALISATION_CPPS})
  endif(VIENNALS_STATIC_BUILD)

  target_link_libraries(${PROJECT_NAME} ${VIENNALS_LIBRARIES})
  target_include_directories(${PROJECT_NAME} PRIVATE "${PROJECT_SOURCE_DIR}/include/"
                                                     ${VIENNAHRLE_INCLUDE_DIRS})
  set_target_properties(${PROJECT_NAME} PROPERTIES VERSION ${PROJECT_VERSION})
else(VIENNALS_BUILD_SHARED_LIBS)
  add_library(${PROJECT_NAME} INTERFACE)
endif(VIENNALS_BUILD_SHARED_LIBS)

# set the correct paths for installation
if(CMAKE_INSTALL_PREFIX_INITIALIZED_TO_DEFAULT)
  set(CMAKE_INSTALL_PREFIX "${CMAKE_INSTALL_PREFIX}/${PROJECT_NAME}/")
endif()
set(LOCAL_CONFIG_DIR "lib/cmake/${PROJECT_NAME}")

# This variable is used if ViennaLS_COMMON_TARGET is set to true. And should only be invoked for the
# example and test target, since those are compiled before the project is installed in its proper
# install location.
set(VIENNALS_BUILD_INCLUDE_DIRS "${${PROJECT_NAME}_SOURCE_DIR}/include")

# Adding the install interface generator expression makes sure that the include files are installed
# to the proper location (provided by GNUInstallDirs)
target_include_directories(
  ${PROJECT_NAME} INTERFACE $<BUILD_INTERFACE:${VIENNALS_BUILD_INCLUDE_DIRS}>
                            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)

target_compile_features(${PROJECT_NAME} INTERFACE cxx_std_11)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
  "${PROJECT_NAME}ConfigVersion.cmake"
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY AnyNewerVersion)

configure_package_config_file(
  "${PROJECT_SOURCE_DIR}/cmake/${PROJECT_NAME}Config.cmake.in"
  "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake" INSTALL_DESTINATION ${LOCAL_CONFIG_DIR})

# ##################################################################################################
# BUILD EXAMPLES
# ##################################################################################################
option(VIENNALS_BUILD_EXAMPLES "Build examples." OFF)
if(VIENNALS_BUILD_EXAMPLES)
  add_subdirectory(Examples)
endif(VIENNALS_BUILD_EXAMPLES)

# ##################################################################################################
# BUILD TESTS (Smaller examples for testing)
# ##################################################################################################
option(VIENNALS_BUILD_TESTS "Build tests." OFF)
if(VIENNALS_BUILD_TESTS)
  enable_testing()
  add_subdirectory(Tests)
endif(VIENNALS_BUILD_TESTS)

# ##################################################################################################
# BUILD PYTHON MODULE
# ##################################################################################################
# if(VIENNALS_BUILD_PYTHON_2 OR VIENNALS_BUILD_PYTHON_3)
#   add_subdirectory(Wrapping)
# endif()

# ##################################################################################################
# INSTALL
# ##################################################################################################
# install config files locations are provided by GNUInstallDirs
install(
  TARGETS ${PROJECT_NAME}
  EXPORT ${PROJECT_NAME}_Targets
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR})

install(
  EXPORT ${PROJECT_NAME}_Targets
  FILE ${PROJECT_NAME}Targets.cmake
  DESTINATION ${LOCAL_CONFIG_DIR})

install(FILES "${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
              "${PROJECT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake"
        DESTINATION ${LOCAL_CONFIG_DIR})

# install include files
file(GLOB_RECURSE HEADER_FILES "${PROJECT_SOURCE_DIR}/include/*.hpp")
install(FILES ${HEADER_FILES} DESTINATION include)
