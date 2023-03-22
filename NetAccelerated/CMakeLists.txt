# CMake setup
cmake_minimum_required (VERSION 3.12)
MESSAGE(STATUS "CMAKE_ROOT: " ${CMAKE_ROOT})

# SET THE COMPILER
set(CMAKE_CXX_COMPILER /opt/nvidia/hpc_sdk/Linux_x86_64/22.9/compilers/bin/nvc++)

# Project name
project(OpenTimerAccelerated)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(OPENACC_PATH /opt/nvidia/hpc_sdk/Linux_x86_64/22.11/cuda/11.8)
set(OPENACC_INC "-I${OPENACC_PATH}/include")
set(OPENACC_LIB "-L${OPENACC_PATH}/lib64 -Wl,-R${OPENACC_PATH}/lib64 -lcudart")

set(ACCELERATED_LIBRARIES "${OPENACC_INC} -L${OPENACC_LIB} -O2 -fast -gpu=nordc -acc -Minfo=accel -std=c++17 -lstdc++ -lcudart")

set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${ACCELERATED_LIBRARIES} -Wall -fPIC")

# Check if static build
if(CMAKE_EXE_LINKER_FLAGS)
  string(FIND ${CMAKE_EXE_LINKER_FLAGS} "-static" IS_STATIC)
  if(NOT ${IS_STATIC} EQUAL -1)
    set(CMAKE_EXE_LINKER_FLAGS "${CMAKE_EXE_LINKER_FLAGS} -Wl,--whole-archive ${CMAKE_THREAD_LIBS_INIT} -Wl,--no-whole-archive")
  endif()
endif()

set(CMAKE_INSTALL_PREFIX ${CMAKE_CURRENT_LIST_DIR}/library)

# Set library output path
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/lib)
message(STATUS "CMAKE_ARCHIVE_OUTPUT_DIRECTORY: ${PROJECT_SOURCE_DIR}/lib")

###########################################################
# Library dependencies
###########################################################

# Find program
find_program(PYTHON_EXECUTABLE python)
message(STATUS "PYTHON_EXECUTABLE: ${PYTHON_EXECUTABLE}")

# Find package
find_package(Threads REQUIRED)

# Tclsh module
include(FindTclsh)
if (NOT TCL_TCLSH)
  message(FATAL_ERROR "\n\
OpenTimer requires tclsh to build. Visit: https://www.tcl.tk/software/tcltk/")
endif(NOT TCL_TCLSH)

# CTest
include(CTest)

# configure a header file to pass some of the CMake settins
configure_file(
  "${PROJECT_SOURCE_DIR}/../ot/config.hpp.in"
  "${PROJECT_SOURCE_DIR}/../ot/config.hpp"
)

# add the binary tree to the search path for include files so we can find TutorialConfig.h
include_directories(${PROJECT_SOURCE_DIR}/../)

# Net source
set(OT_NET
  ${CMAKE_CURRENT_LIST_DIR}/../ot/timer/net.cpp)

# Add OpenTimer library
add_library(NetAccelerated STATIC ${OT_NET})
target_include_directories(NetAccelerated PUBLIC ${CMAKE_CURRENT_LIST_DIR}/../)
install(TARGETS NetAccelerated
  DESTINATION ${CMAKE_INSTALL_PREFIX})


