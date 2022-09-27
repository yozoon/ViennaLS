cmake_minimum_required(VERSION 3.4)

project(ViennaLSExamples)

include("${CMAKE_SOURCE_DIR}/cmake/preamble.cmake")

subdirlist(EXAMPLE_DIRS ${CMAKE_CURRENT_LIST_DIR})

# add all examples
foreach(subdir ${EXAMPLE_DIRS})
  add_subdirectory(${subdir})
endforeach()
