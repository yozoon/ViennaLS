cmake_minimum_required(VERSION 3.8 FATAL_ERROR)

project(ViennaLSTests)

include("${CMAKE_SOURCE_DIR}/cmake/prepare.cmake")

add_custom_target(buildTests DEPENDS ${VIENNALS_DEPENDENCIES})

subdirlist(TEST_DIRS ${CMAKE_CURRENT_LIST_DIR})

# add all tests
foreach(subdir ${TEST_DIRS})
  add_subdirectory(${subdir})
endforeach()
