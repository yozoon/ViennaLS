cmake_minimum_required(VERSION 3.8 FATAL_ERROR)

project(ViennaLSPrecompiled)

# Point find_package to the binary directory instead of the install location
find_package(ViennaLS CONFIG PATHS ${ViennaLS_BINARY_DIR})
if(NOT ViennaLS_FOUND)
  message(
    FATAL_ERROR
      "One of the dependencies hasn't been built yet and thus can't be used by the precompiled libraries target.\n\
Try the following steps to fix the problem:\n\
1. Turn off precompiled libraries target by disabling the VIENNALS_BUILD_SHARED_LIBS option when configuring the project\n\
   (e.g. with the `-D VIENNALS_BUILD_SHARED_LIBS=OFF` flag)\n\
2. Build the dependencies target: e.g. `make buildDependencies` when using Makefiles\n\
3. Turn on precompiled libraries target again by re-enabling the VIENNALS_BUILD_SHARED_LIBS option\n\
   (e.g. with the `-D VIENNALS_BUILD_SHARED_LIBS=ON` flag)\n\
4. Now the configure step should pick up the built dependencies and proceed as intended.")
endif()

list(PREPEND VIENNALS_INCLUDE_DIRS ${VIENNALS_BUILD_INCLUDE_DIRS})
list(APPEND VIENNALS_LIBRARIES ViennaHRLE)

# add all specialisation source files
file(GLOB SPECIALISATION_CPPS "${CMAKE_CURRENT_LIST_DIR}/*.cpp")

# build static or shared object for library
if(VIENNALS_STATIC_BUILD)
  add_library(${PROJECT_NAME} STATIC ${SPECIALISATION_CPPS})
else(VIENNALS_STATIC_BUILD)
  add_library(${PROJECT_NAME} SHARED ${SPECIALISATION_CPPS})
endif(VIENNALS_STATIC_BUILD)

target_link_libraries(${PROJECT_NAME} ${VIENNALS_LIBRARIES})

target_include_directories(${PROJECT_NAME} PRIVATE ${VIENNALS_INCLUDE_DIRS})

set_target_properties(${PROJECT_NAME} PROPERTIES VERSION ${VIENNALS_VERSION})

# Building the shared libraries requires that all dependencies are built
add_dependencies(${PROJECT_NAME} ${VIENNALS_DEPENDENCIES})

include(GNUInstallDirs)

# Adding the install interface generator expression makes sure that the include files are installed
# to the proper location (provided by GNUInstallDirs)
target_include_directories(
  ${PROJECT_NAME} INTERFACE $<BUILD_INTERFACE:${VIENNALS_BUILD_INCLUDE_DIRS}>
                            $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>)

target_compile_features(${PROJECT_NAME} INTERFACE cxx_std_11)

install(TARGETS ${PROJECT_NAME} LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})
