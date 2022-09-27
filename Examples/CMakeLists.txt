cmake_minimum_required(VERSION 3.8 FATAL_ERROR)

project(ViennaLSExamples)

include("${CMAKE_SOURCE_DIR}/cmake/prepare.cmake")

add_custom_target(buildExamples DEPENDS ${VIENNALS_DEPENDENCIES})

subdirlist(EXAMPLE_DIRS ${CMAKE_CURRENT_LIST_DIR})

# add all examples
foreach(subdir ${EXAMPLE_DIRS})
  add_subdirectory(${subdir})
endforeach()
