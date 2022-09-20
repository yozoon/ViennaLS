## external project base directory for all projects in this hierarchy
set_property(DIRECTORY PROPERTY EP_BASE ${DEPENDENCIES_DIR})

if(VIENNALS_USE_VTK)
    # if no directory is specified, or this is a re-run
    if(NOT VTK_FOUND)
        add_subdirectory(vtk)
        message(STATUS "VTK_DIR: ${VTK_DIR}")
    endif()
endif()

# build viennahrle if no path was specified
if(NOT ViennaHRLE_FOUND)
    add_subdirectory(viennahrle)
    message(STATUS "ViennaHRLE_DIR: ${ViennaHRLE_DIR}")
endif()

# build pybind11 if no path was specified
# if(NOT pybind11_FOUND)
#     add_subdirectory(pybind11)
#     message(STATUS "pybind11_DIR: ${pybind11_DIR}")
# endif()