# ##################################################################################################
# Check VTK Dependency
# ##################################################################################################
if(DEFINED ${VTK_DIR})
  find_package(
    VTK CONFIG REQUIRED PATHS ${VTK_DIR}
    COMPONENTS vtkCommonCore
               vtkIOCore
               vtkIOXML
               vtkIOXMLParser
               vtkFiltersCore
               vtkFiltersGeneral
               vtkFiltersGeometry)
  message(STATUS "VTK_DIR: ${VTK_DIR}")
  add_library(vtk_external INTERFACE) # dummy target
else()
  find_package(
    VTK QUIET
    # COMPONENTS VTK::CommonCore VTK::IOCore VTK::IOXML VTK::IOXMLParser VTK::FiltersCore
    # VTK::FiltersGeneral VTK::FiltersGeometry
  )
  if(VTK_FOUND)
    message(STATUS "Found VTK at ${VTK_DIR}.")
    add_library(vtk_external INTERFACE) # dummy target
  else()
    message(
      STATUS
        "Couldn't find VTK. Dependency will be built with the buildDependencies target (e.g. `make buildDependencies`)."
    )
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

    set(VTK_VERSION 9.1)

    ExternalProject_Add(
      vtk_external
      GIT_REPOSITORY https://gitlab.kitware.com/vtk/vtk.git
      # Hash of tags/v9.1.0
      GIT_TAG # v9.1.0
              285daeedd58eb890cb90d6e907d822eea3d2d092
      GIT_SHALLOW true
      GIT_SUBMODULES ""
      TMP_DIR "${DEPENDENCIES_DIR}/tmp/${VTK_EXTERNAL_INSTALL}"
      STAMP_DIR "${DEPENDENCIES_DIR}/Stamp/${VTK_EXTERNAL_INSTALL}"
      BINARY_DIR "${DEPENDENCIES_DIR}/Build/${VTK_EXTERNAL_INSTALL}"
      CMAKE_ARGS -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR>
                 -DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}
                 -DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}
                 -DCMAKE_CXX_STANDARD=${CMAKE_CXX_STANDARD}
                 -DCMAKE_CXX_EXTENSIONS=${CMAKE_CXX_EXTENSIONS}
                 -DCMAKE_CXX_STANDARD_REQUIRED=${CMAKE_CXX_STANDARD_REQUIRED}
                 -DBUILD_SHARED_LIBS:BOOL=${VTK_EXTERNAL_BUILD_SHARED}
                 -DVTK_BUILD_TESTING=OFF
                 -DVTK_BUILD_EXAMPLES=OFF
                 -DVTK_BUILD_DOCUMENTATION=OFF
                 -DVTK_WRAP_PYTHON:BOOL=NO
                 -DVTK_SMP_IMPLEMENTATION_TYPE:STRING="OpenMP"
                 -DVTK_GROUP_ENABLE_Rendering:BOOL=NO
                 -DVTK_GROUP_ENABLE_Imaging:BOOL=NO
                 -DVTK_GROUP_ENABLE_MPI:BOOL=NO
                 -DVTK_GROUP_ENABLE_Qt:BOOL=NO
                 -DVTK_GROUP_ENABLE_StandAlone:BOOL=NO
                 -DVTK_GROUP_ENABLE_Views:BOOL=NO
                 -DVTK_GROUP_ENABLE_Web:BOOL=NO
                 -DVTK_LEGACY_REMOVE:BOOL=ON
                 -DVTK_MODULE_ENABLE_VTK_CommonExecutionModel:BOOL=YES
                 -DVTK_MODULE_ENABLE_VTK_CommonMisc:BOOL=YES
                 -DVTK_MODULE_ENABLE_VTK_CommonSystem:BOOL=YES
                 -DVTK_MODULE_ENABLE_VTK_CommonMath:BOOL=YES
                 -DVTK_MODULE_ENABLE_VTK_CommonCore:BOOL=YES
                 -DVTK_MODULE_ENABLE_VTK_CommonTransforms:BOOL=YES
                 -DVTK_MODULE_ENABLE_VTK_CommonComputationalGeometry:BOOL=YES
                 -DVTK_MODULE_ENABLE_VTK_IOCore:BOOL=YES
                 -DVTK_MODULE_ENABLE_VTK_IOXMLParser:BOOL=YES
                 -DVTK_MODULE_ENABLE_VTK_IOXML:BOOL=YES
                 -DVTK_MODULE_ENABLE_VTK_FiltersCore:BOOL=YES
                 -DVTK_MODULE_ENABLE_VTK_FiltersGeneral:BOOL=YES
                 -DVTK_MODULE_ENABLE_VTK_FiltersGeometry:BOOL=YES
      BUILD_COMMAND ${CMAKE_COMMAND} --build . --parallel ${NPROC}
      # No install needed
      INSTALL_COMMAND ""
      UPDATE_COMMAND ""
      USES_TERMINAL_DOWNLOAD 1
      USES_TERMINAL_UPDATE 1
      GIT_PROGRESS 1)

    # Get install and build directory
    ExternalProject_Get_Property(vtk_external INSTALL_DIR BINARY_DIR)

    # Since we don't explicitely install vtk (INSTALL_COMMAND is empty), we simply provide the build
    # directory as a path for looking up this installation.
    set(VTK_DIR "${BINARY_DIR}")
  endif()
  # Write VTK_DIR and VTK_VERSION to the cache so that we can use it in the toplevel CMakeLists file
  set(VTK_DIR
      ${VTK_DIR}
      CACHE PATH "Path to VTK installation" FORCE)
  set(VTK_VERSION
      ${VTK_VERSION}
      CACHE STRING "The VTK version" FORCE)
endif()
