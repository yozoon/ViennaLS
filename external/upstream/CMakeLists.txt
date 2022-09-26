# OpenMP Support is required
find_package(OpenMP REQUIRED)

if(VIENNALS_USE_VTK)
  add_subdirectory(vtk)
endif()

add_subdirectory(viennahrle)

add_subdirectory(pybind11)
