# Copyright (c) 2014-2015, CosmicPy Developers
# Licensed under CeCILL 2.1 - see LICENSE.rst
cmake_minimum_required(VERSION 2.8)

include(ExternalProject)
include(FindPkgConfig)

# Adding customized cmake module
list(APPEND CMAKE_MODULE_PATH  "${CMAKE_SOURCE_DIR}/cmake/Modules/")

project(starlet)

# Find required packages
find_package(Boost REQUIRED COMPONENTS python)
find_package(PythonInterp REQUIRED)
find_package(PythonLibsNew REQUIRED)

pkg_check_modules(GSL REQUIRED gsl)
pkg_check_modules(CFITSIO REQUIRED cfitsio)

# Downloads and compiles the Boost.NumPy package
# The library and include files are located in the build/extern directory
ExternalProject_Add(BoostNumpy
    GIT_REPOSITORY https://github.com/ndarray/Boost.NumPy
    CMAKE_ARGS     -DCMAKE_INSTALL_PREFIX=${CMAKE_BINARY_DIR}/extern
                   -DLIBRARY_TYPE=STATIC
)
set(BoostNumpy_INCLUDE_DIRS ${CMAKE_BINARY_DIR}/extern/include)
set(BoostNumpy_LIBRARY_DIRS ${CMAKE_BINARY_DIR}/extern/lib64)
set(BoostNumpy_LIBRARIES -lboost_numpy)


# Define include and library directories
include_directories(
  ${GSL_INCLUDE_DIRS}
  ${PYTHON_INCLUDE_DIRS}
  ${Boost_INCLUDE_DIRS}
  ${BoostNumpy_INCLUDE_DIRS}
  ${CFITSIO_INCLUDE_DIRS}
)
link_directories(${BoostNumpy_LIBRARY_DIRS} ${GSL_LIBRARY_DIRS} ${CFITSIO_LIBRARY_DIRS})

# Compilation flags
#if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "Clang")
  # using Clang, disabling OpenMP support
#  set(CMAKE_CXX_FLAGS "-O3 -fomit-frame-pointer -fno-common -fPIC")
#else()
#  set(CMAKE_CXX_FLAGS "-O3 -fomit-frame-pointer -fno-common -fPIC -fopenmp")
#endif()

set(CMAKE_CXX_FLAGS "-O3 -fomit-frame-pointer -fno-common -fPIC -fopenmp -DRW_PYTHON_INTERFACE -F/Library/Frameworks -arch x86_64")


# Build the tools module
add_library(starlet SHARED src/starlet_Interface.cpp src/starlet.cpp)
add_dependencies(starlet BoostNumpy)
target_link_libraries(starlet ${BoostNumpy_LIBRARIES} ${Boost_LIBRARIES} ${GSL_LIBRARIES} ${PYTHON_LIBRARIES})
set_target_properties(starlet PROPERTIES SUFFIX .so)
set_target_properties(starlet PROPERTIES PREFIX "")


