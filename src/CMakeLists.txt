cmake_minimum_required(VERSION 3.6 FATAL_ERROR)

project(ViennaLS_core)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_CXX_STANDARD "17")

include(GNUInstallDirs)

find_package(OpenMP REQUIRED)
list(APPEND VIENNALS_LIBRARIES OpenMP::OpenMP_CXX)

find_package(ViennaHRLE REQUIRED)
if(VIENNALS_USE_VTK)
  find_package(
    VTK REQUIRED
    COMPONENTS VTK::CommonCore
               VTK::IOCore
               VTK::IOXML
               VTK::IOXMLParser
               VTK::FiltersCore
               VTK::FiltersGeneral
               VTK::FiltersGeometry)

  add_compile_definitions(VIENNALS_USE_VTK)

  # use VTK_USE_FILE for older versions of VTK
  if(VTK_VERSION VERSION_LESS "8.90")
    message(STATUS "Using old VTK Version: ${VTK_VERSION}")
    set(VTK_LIBRARIES
        vtksys;vtkIOCore;vtkexpat;vtklz4;vtkzlib;vtklzma;vtkdoubleconversion;vtkCommonMisc;vtkCommonSystem;vtkIOXML;vtkFiltersCore;vtkFiltersGeneral;vtkFiltersGeometry;vtkCommonExecutionModel;vtkCommonDataModel;vtkCommonColor;vtkIOXMLParser;vtkIOCore;vtkCommonMisc;vtkCommonComputationalGeometry;vtkCommonTransforms;vtkCommonMath;
    )
    include(${VTK_USE_FILE})
  else()
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
  # list(APPEND VIENNALS_LIBRARIES ViennaHRLE) find_package(ViennaHRLE REQUIRED PATHS
  # ${ViennaHRLE_DIR})

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

# Adding the install interface generator expression makes sure that the include files are installed
# to the proper location (provided by GNUInstallDirs)
set(VIENNALS_BUILD_INCLUDE_DIRS "${CMAKE_INSTALL_PREFIX}/include")
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
option(VIENNALS_BUILD_PYTHON_2 "Build for python2.x." OFF)
option(VIENNALS_BUILD_PYTHON_3 "Build for python3.x. Trumps python2.x build." OFF)

if(VIENNALS_BUILD_PYTHON_2 OR VIENNALS_BUILD_PYTHON_3)
  # include external project if not already set
  if(NOT DEFINED pybind11_DIR OR VIENNALS_BUILD_PYBIND11)
    add_subdirectory(external/upstream/pybind11)
  endif()

  message(STATUS "pybind11_DIR: ${pybind11_DIR}")
  find_package(pybind11 QUIET PATHS ${pybind11_DIR})
  if(pybind11_FOUND)
    add_subdirectory(Wrapping)
  else()
    message(
      STATUS
        "No pybind11 install found. If you did not specify a custom install path, you have to build the target pybind11-external."
    )
  endif()

endif()

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
