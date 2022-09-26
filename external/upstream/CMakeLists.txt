# OpenMP Support is required
find_package(OpenMP REQUIRED)

if(VIENNALS_USE_VTK)
  add_subdirectory(vtk)
endif()

add_subdirectory(viennahrle)

if(VIENNALS_BUILD_PYTHON_2 OR VIENNALS_BUILD_PYTHON_3)
  add_subdirectory(pybind11)
endif()
