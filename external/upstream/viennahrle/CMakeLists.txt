externalproject_add(viennahrle-external
  GIT_REPOSITORY
  https://github.com/ViennaTools/ViennaHRLE
  GIT_SHALLOW 1 # use --depth 1 when cloning
  CMAKE_ARGS
    -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR>
    -DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}
    -DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}
    -DCMAKE_CXX_STANDARD=${CMAKE_CXX_STANDARD}
    -DCMAKE_CXX_EXTENSIONS=${CMAKE_CXX_EXTENSIONS}
    -DCMAKE_CXX_STANDARD_REQUIRED=${CMAKE_CXX_STANDARD_REQUIRED}
    -DVIENNAHRLE_BUILD_EXAMPLES=OFF
    -DVIENNAHRLE_BUILD_TESTS=OFF
  USES_TERMINAL_DOWNLOAD
    1
  USES_TERMINAL_UPDATE
    1
  GIT_PROGRESS
    1
  EXCLUDE_FROM_ALL TRUE
)

# Get install and build directory
externalproject_get_property(viennahrle-external INSTALL_DIR BINARY_DIR)

set(
    ViennaHRLE_DIR "${INSTALL_DIR}"
  CACHE PATH "Path to ViennaHRLE installation"
  FORCE
  )

set(
  ViennaHRLE_INCLUDEDIR "${ViennaHRLE_DIR}/include"
  CACHE PATH "Path to ViennaHRLE include directories"
  FORCE
  )

set(
  ViennaHRLE_LIBRARYDIR "${ViennaHRLE_DIR}/lib"
  CACHE PATH "Path to ViennaHRLE library directories"
  FORCE
  )
  
# add to buildDependencies target
add_dependencies(buildDependencies viennahrle-external)
