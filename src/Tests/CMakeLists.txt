cmake_minimum_required(VERSION 3.4)

project(ViennaLSTests)

# Enable Clang sanitizer for debug builds
if("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
  set(CMAKE_CXX_FLAGS_DEBUG
      "${CMAKE_CXX_FLAGS_DEBUG} -fno-omit-frame-pointer -fsanitize=address -fsanitize=thread -fsanitize=memory"
      CACHE STRING "")
  set(CMAKE_EXE_LINKER_FLAGS_DEBUG
      "${CMAKE_EXE_LINKER_FLAGS_DEBUGS} -fno-omit-frame-pointer -fsanitize=address -fsanitize=thread -fsanitize=memory"
      CACHE STRING "")
endif()

macro(SUBDIRLIST result curdir)
  file(
    GLOB children
    RELATIVE ${curdir}
    ${curdir}/*)
  set(dirlist "")
  foreach(child ${children})
    if(IS_DIRECTORY ${curdir}/${child})
      if(NOT ${child} STREQUAL "build")
        list(APPEND dirlist ${child})
      endif()
    endif()
  endforeach()
  set(${result} ${dirlist})
endmacro()

add_custom_target(buildTests)

set(ViennaLS_DIR "${ViennaLS_BINARY_DIR}")
find_package(ViennaLS CONFIG REQUIRED PATHS ${ViennaLS_DIR})
set(ViennaLS_COMMON_TARGET TRUE)

subdirlist(EXAMPLE_DIRS ${CMAKE_CURRENT_LIST_DIR})

# add all examples
foreach(subdir ${EXAMPLE_DIRS})
  add_subdirectory(${subdir})
endforeach()
