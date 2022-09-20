# if this is a static build, build vtk statically
if(VIENNALS_STATIC_BUILD)
  set(VTK_EXTERNAL_BUILD_SHARED OFF)
  set(VTK_EXTERNAL_INSTALL "static-vtk-external")
  message(STATUS "Static build: Using static VTK libs.")
else()
  set(VTK_EXTERNAL_BUILD_SHARED ON)
  set(VTK_EXTERNAL_INSTALL "vtk-external")
endif()

# Get the number of cores
include(ProcessorCount)
ProcessorCount(NPROC)

externalproject_add(vtk-external
  GIT_REPOSITORY
    https://gitlab.kitware.com/vtk/vtk.git
  # Hash of tags/v9.1.0
  GIT_TAG #v9.1.0
    285daeedd58eb890cb90d6e907d822eea3d2d092
  GIT_SHALLOW true
  GIT_SUBMODULES ""
  TMP_DIR
    "${DEPENDENCIES_DIR}/tmp/${VTK_EXTERNAL_INSTALL}"
  STAMP_DIR
    "${DEPENDENCIES_DIR}/Stamp/${VTK_EXTERNAL_INSTALL}"
  BINARY_DIR
    "${DEPENDENCIES_DIR}/Build/${VTK_EXTERNAL_INSTALL}"
  CMAKE_ARGS
    -D CMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR>
    -D CMAKE_C_COMPILER=${CMAKE_C_COMPILER}
    -D CMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}
    -D CMAKE_CXX_STANDARD=${CMAKE_CXX_STANDARD}
    -D CMAKE_CXX_EXTENSIONS=${CMAKE_CXX_EXTENSIONS}
    -D CMAKE_CXX_STANDARD_REQUIRED=${CMAKE_CXX_STANDARD_REQUIRED}
    -D BUILD_SHARED_LIBS:BOOL=${VTK_EXTERNAL_BUILD_SHARED}
    -D VTK_BUILD_TESTING=OFF
    -D VTK_BUILD_EXAMPLES=OFF
    -D VTK_BUILD_DOCUMENTATION=OFF
    -D VTK_WRAP_PYTHON:BOOL=NO
    -D VTK_SMP_IMPLEMENTATION_TYPE:STRING="OpenMP"
    -D VTK_GROUP_ENABLE_Rendering:BOOL=NO
    -D VTK_GROUP_ENABLE_Imaging:BOOL=NO
    -D VTK_GROUP_ENABLE_MPI:BOOL=NO
    -D VTK_GROUP_ENABLE_Qt:BOOL=NO
    -D VTK_GROUP_ENABLE_StandAlone:BOOL=NO
    -D VTK_GROUP_ENABLE_Views:BOOL=NO
    -D VTK_GROUP_ENABLE_Web:BOOL=NO
    -D VTK_LEGACY_REMOVE:BOOL=ON
    -D VTK_MODULE_ENABLE_VTK_CommonExecutionModel:BOOL=YES
    -D VTK_MODULE_ENABLE_VTK_CommonMisc:BOOL=YES
    -D VTK_MODULE_ENABLE_VTK_CommonSystem:BOOL=YES
    -D VTK_MODULE_ENABLE_VTK_CommonMath:BOOL=YES
    -D VTK_MODULE_ENABLE_VTK_CommonCore:BOOL=YES
    -D VTK_MODULE_ENABLE_VTK_CommonTransforms:BOOL=YES
    -D VTK_MODULE_ENABLE_VTK_CommonComputationalGeometry:BOOL=YES
    -D VTK_MODULE_ENABLE_VTK_IOCore:BOOL=YES
    -D VTK_MODULE_ENABLE_VTK_IOXMLParser:BOOL=YES
    -D VTK_MODULE_ENABLE_VTK_IOXML:BOOL=YES
    -D VTK_MODULE_ENABLE_VTK_FiltersCore:BOOL=YES
    -D VTK_MODULE_ENABLE_VTK_FiltersGeneral:BOOL=YES
    -D VTK_MODULE_ENABLE_VTK_FiltersGeometry:BOOL=YES
  BUILD_COMMAND ${CMAKE_COMMAND} --build . --parallel ${NPROC}
  # No install needed
  INSTALL_COMMAND
    ""
  UPDATE_COMMAND
    ""
  USES_TERMINAL_DOWNLOAD
    1
  USES_TERMINAL_UPDATE
    1
  GIT_PROGRESS
    1
  EXCLUDE_FROM_ALL TRUE
)

# Get install and build directory
externalproject_get_property(vtk-external INSTALL_DIR BINARY_DIR)

# Since we don't explicitely install vtk (INSTALL_COMMAND is empty),
# we simply provide the build directory as a path for looking up this
# installation.
set(VTK_DIR "${BINARY_DIR}")
set(
  VTK_DIR ${VTK_DIR}
  CACHE PATH "Path to VTK installation" FORCE)

# add to buildDependencies target
add_dependencies(buildDependencies vtk-external)
