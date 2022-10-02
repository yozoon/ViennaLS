# Macro to check if all of the libraries provided by the target list LIBRARY_TARGETS are compiled as
# static libraries
macro(CHECK_ALL_STATIC RESULT LIBRARY_TARGETS)
  set(_all_static TRUE)
  foreach(_lib ${LIBRARY_TARGETS})
    if(TARGET ${_lib})
      get_target_property(_target_type ${_lib} TYPE)
      # message(STATUS "${_lib}: ${_target_type}")
      if(NOT _target_type STREQUAL "STATIC_LIBRARY")
        set(_all_static FALSE)
        break()
      endif()
    else()
      message(WARNING "Not all targets were found!")
      set(_all_static FALSE)
      break()
    endif()
  endforeach()
  set(${RESULT} ${_all_static})
endmacro()

# OpenMP Support is required
find_package(OpenMP REQUIRED)

if(VIENNALS_USE_VTK)
  add_subdirectory(vtk)
endif()

add_subdirectory(viennahrle)

if(VIENNALS_BUILD_PYTHON_2 OR VIENNALS_BUILD_PYTHON_3)
  add_subdirectory(pybind11)
endif()
