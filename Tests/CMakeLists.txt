cmake_minimum_required(VERSION 3.4)

project(ViennaLSTests)

include("${CMAKE_SOURCE_DIR}/cmake/preamble.cmake")

add_custom_target(buildTests DEPENDS ${VIENNALS_DEPENDENCIES})

subdirlist(EXAMPLE_DIRS ${CMAKE_CURRENT_LIST_DIR})

# add all examples
foreach(subdir ${EXAMPLE_DIRS})
  add_subdirectory(${subdir})
endforeach()
