# target for python module build

# Get the number of cores
include(ProcessorCount)
ProcessorCount(NPROC)

set(vls_python_external pythonBindings)
ExternalProject_Add(
  ${vls_python_external}
  DEPENDS ${VIENNALS_DEPENDENCIES}
  SOURCE_DIR ${CMAKE_CURRENT_LIST_DIR}/Python
  BINARY_DIR ${CMAKE_BINARY_DIR}
  CMAKE_ARGS -DCMAKE_INSTALL_PREFIX=${CMAKE_INSTALL_PREFIX}
             -DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}
             -DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}
             -DCMAKE_CXX_STANDARD=${CMAKE_CXX_STANDARD}
             -DCMAKE_CXX_EXTENSIONS=${CMAKE_CXX_EXTENSIONS}
             -DCMAKE_CXX_STANDARD_REQUIRED=${CMAKE_CXX_STANDARD_REQUIRED}
             -DCMAKE_VERBOSE_MAKEFILE=${CMAKE_VERBOSE_MAKEFILE}
             -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE}
             -DCMAKE_CXX_FLAGS=${CMAKE_CXX_FLAGS}
             -DCMAKE_CXX_FLAGS_DEBUG=${CMAKE_CXX_FLAGS_DEBUG}
             -DCMAKE_CXX_FLAGS_RELEASE=${CMAKE_CXX_FLAGS_RELEASE}
             # Paths
             -DViennaLS_BINARY_DIR=${ViennaLS_BINARY_DIR}
             -DViennaHRLE_DIR=${ViennaHRLE_DIR}
             -DVTK_DIR=${VTK_DIR}
             -Dpybind11_DIR=${pybind11_DIR}
             # Variables and Options
             -DVIENNALS_VERSION=${PROJECT_VERSION}
             -DVIENNALS_BUILD_PYTHON_3=${VIENNALS_BUILD_PYTHON_3}
             -DVIENNALS_BUILD_PYTHON_2=${VIENNALS_BUILD_PYTHON_2}
  CMAKE_CACHE_ARGS
    -DCMAKE_CXX_FLAGS:STRING=${CMAKE_CXX_FLAGS} -DCMAKE_PREFIX_PATH:PATH=${CMAKE_PREFIX_PATH}
    -DCMAKE_INCLUDE_PATH:PATH=${CMAKE_INCLUDE_PATH} -DCMAKE_LIBRARY_PATH:PATH=${CMAKE_LIBRARY_PATH}
  BUILD_ALWAYS 1)

ExternalProject_Add_StepTargets(
  ${vls_python_external}
  build
  DEPENDEES
  configure
  BUILD_COMMAND
  "${CMAKE_COMMAND} --build . --parallel ${NPROC}"
  EXCLUDE_FROM_MAIN
  TRUE)

ExternalProject_Add_StepTargets(${vls_python_external} install DEPENDEES build EXCLUDE_FROM_MAIN
                                TRUE)
