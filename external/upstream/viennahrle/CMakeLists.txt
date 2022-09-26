# ##################################################################################################
# Check ViennaHRLE Dependency
# ##################################################################################################
if(DEFINED ${ViennaHRLE_DIR})
  find_package(ViennaHRLE REQUIRED PATHS ${ViennaHRLE_DIR})
  message(STATUS "ViennaHRLE_DIR: ${ViennaHRLE_DIR}")
  add_library(viennahrle_external INTERFACE) # dummy target
else()
  find_package(ViennaHRLE QUIET)
  if(ViennaHRLE_FOUND)
    message(STATUS "Found ViennaHRLE at ${ViennaHRLE_DIR}.")
    add_library(viennahrle_external INTERFACE) # dummy target
  else()
    message(
      STATUS
        "Couldn't find ViennaHRLE. Dependency will be built with the buildDependencies target (e.g. `make buildDependencies`)."
    )
    ExternalProject_Add(
      viennahrle_external
      GIT_REPOSITORY https://github.com/ViennaTools/ViennaHRLE
      GIT_SHALLOW 1 # use --depth 1 when cloning
      CMAKE_ARGS -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR>
                 -DCMAKE_C_COMPILER=${CMAKE_C_COMPILER}
                 -DCMAKE_CXX_COMPILER=${CMAKE_CXX_COMPILER}
                 -DCMAKE_CXX_STANDARD=${CMAKE_CXX_STANDARD}
                 -DCMAKE_CXX_EXTENSIONS=${CMAKE_CXX_EXTENSIONS}
                 -DCMAKE_CXX_STANDARD_REQUIRED=${CMAKE_CXX_STANDARD_REQUIRED}
                 -DVIENNAHRLE_BUILD_EXAMPLES=OFF
                 -DVIENNAHRLE_BUILD_TESTS=OFF
      USES_TERMINAL_DOWNLOAD 1
      USES_TERMINAL_UPDATE 1
      GIT_PROGRESS 1)

    # Get install and build directory
    ExternalProject_Get_Property(viennahrle_external INSTALL_DIR BINARY_DIR)

    set(ViennaHRLE_DIR
        "${INSTALL_DIR}"
        CACHE PATH "Path to ViennaHRLE installation" FORCE)

    set(ViennaHRLE_INCLUDEDIR
        "${ViennaHRLE_DIR}/include"
        CACHE PATH "Path to ViennaHRLE include directories" FORCE)

    set(ViennaHRLE_LIBRARYDIR
        "${ViennaHRLE_DIR}/lib"
        CACHE PATH "Path to ViennaHRLE library directories" FORCE)

  endif()
endif()
