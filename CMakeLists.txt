cmake_minimum_required(VERSION 3.18)
include(CMakeDependentOption)

option(CMAKE_VERBOSE_MAKEFILE "Create verbose makefile" ON)
option(CUDA_VERBOSE_BUILD "Create verbose CUDA build" ON)
option(ENABLE_CUDA "Enable CUDA support" ON)

if(ENABLE_VCPKG_INTEGRATION AND DEFINED ENV{VCPKG_ROOT} AND NOT DEFINED CMAKE_TOOLCHAIN_FILE)
  set(CMAKE_TOOLCHAIN_FILE "$ENV{VCPKG_ROOT}/scripts/buildsystems/vcpkg.cmake" CACHE STRING "")
  message(STATUS "VCPKG found: $ENV{VCPKG_ROOT}")
  message(STATUS "Using VCPKG integration")
  set(USE_INTEGRATED_LIBS "FALSE" CACHE BOOL "Use libs distributed with this repo")
elseif(DEFINED CMAKE_TOOLCHAIN_FILE)
  message(STATUS "Using toolchain: ${CMAKE_TOOLCHAIN_FILE}")
  set(USE_INTEGRATED_LIBS "FALSE" CACHE BOOL "Use libs distributed with this repo")
elseif(WIN32)
  message(STATUS "vcpkg not found, toolchain not defined, using integrated libs on win32")
  set(USE_INTEGRATED_LIBS "TRUE" CACHE BOOL "Use libs distributed with this repo")
else()
  message(WARNING "vcpkg not found, toolchain not defined, system not win32 so build might fail")
  set(USE_INTEGRATED_LIBS "FALSE" CACHE BOOL "Use libs distributed with this repo")
endif()

project(GHActionsTests)

enable_language(C)
enable_language(CXX)

if(ENABLE_CUDA)
  include(CheckLanguage)
  message(STATUS "CMAKE_CUDA_ARCHITECTURES pre-check_language: ${CMAKE_CUDA_ARCHITECTURES}")
  check_language(CUDA)
  message(STATUS "CMAKE_CUDA_ARCHITECTURES post-check_language: ${CMAKE_CUDA_ARCHITECTURES}")
  if(NOT CMAKE_CUDA_COMPILER)
    message(STATUS "PATH: $ENV{PATH}")
    message(STATUS "CUDA_PATH: $ENV{CUDA_PATH}")
    message(STATUS "CUDA_TOOLKIT_ROOT_DIR: $ENV{CUDA_TOOLKIT_ROOT_DIR}")
    message(STATUS "CUDACXX: $ENV{CUDACXX}")
    if(EXISTS "${CUDACXX}")
      message(STATUS "CUDACXX executable found but strangely not accepted")
    else()
      message(STATUS "CUDACXX env variable cannot be resolved on filesystem")
    endif()
  else()
    enable_language(CUDA)
    message(STATUS "Final CMAKE_CUDA_ARCHITECTURES: ${CMAKE_CUDA_ARCHITECTURES}")
  endif()
endif()

if(EXISTS "${CMAKE_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/CMakeOutput.log")
  file(READ ${CMAKE_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/CMakeOutput.log CMAKE_OUTPUT_FILE)
endif()

if(EXISTS "${CMAKE_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/CMakeError.log")
  file(READ ${CMAKE_BINARY_DIR}${CMAKE_FILES_DIRECTORY}/CMakeError.log CMAKE_ERROR_FILE)
endif()

message(STATUS "CMakeOutput.log\n${CMAKE_OUTPUT_FILE}")
message(STATUS "CMakeError.log\n${CMAKE_ERROR_FILE}")

add_executable(prova_cuda cuda_test.cpp)
