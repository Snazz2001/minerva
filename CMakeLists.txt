cmake_minimum_required(VERSION 2.8)

project(Minerva)

message(STATUS "cmake generator: ${CMAKE_GENERATOR}")
message(STATUS "cmake build tool: ${CMAKE_BUILD_TOOL}")
message(STATUS "cmake build type: ${CMAKE_BUILD_TYPE}")

set(COMPILER_FLAGS "-Wall -pthread -m64 -fPIC -march=native")
set(GENERAL_FLAGS_DEBUG "${COMPILER_FLAGS} -O0 -g")
set(GENERAL_FLAGS_RELEASE "${COMPILER_FLAGS} -O2 -mtune=native -mssse3 -ftree-vectorize -funswitch-loops -ftree-vectorizer-verbose=0 -DNDEBUG")
set(CMAKE_CXX_FLAGS_DEBUG "${GENERAL_FLAGS_DEBUG} -std=c++11")
set(CMAKE_CXX_FLAGS_RELEASE "${GENERAL_FLAGS_RELEASE} -std=c++11 -flto")

set(CUDA_TOOLKIT_ROOT_DIR ${CUDA_ROOT})

if (NOT DISABLE_CUDA)
  find_package(CUDA)
  if (CUDA_FOUND)
    message(STATUS "Compiling with CUDA support")
    if (CUDA_VERSION_MAJOR LESS 6)
      message(FATAL_ERROR "Need CUDA 6 for uniform addressing")
    endif (CUDA_VERSION_MAJOR LESS 6)
    include_directories(SYSTEM ${CUDA_INCLUDE_DIRS})
    add_definitions(-DHAS_CUDA)
    string(REPLACE " " "\\\",\\\"" CUDA_CONCAT_DEBUG "${GENERAL_FLAGS_DEBUG}")
    string(REPLACE " " "\\\",\\\"" CUDA_CONCAT_RELEASE "${GENERAL_FLAGS_RELEASE}")
    set(CUDA_PROPAGATE_HOST_FLAGS OFF)
    set(CUDA_NVCC_FLAGS_DEBUG "-G -Xcompiler \\\"${CUDA_CONCAT_DEBUG}\\\"")
    set(CUDA_NVCC_FLAGS_RELEASE "-Xcompiler \\\"${CUDA_CONCAT_RELEASE}\\\"")
    set(CUDA_NVCC_FLAGS "-arch sm_35")
  else ()
    message(FATAL_ERROR "CUDA enabled but not found")
  endif ()
else ()
  message(STATUS "CUDA disabled by user")
endif()

if (NOT DISABLE_PS)
  message(STATUS "Compiling with PS support")
  add_definitions(-DHAS_PS)
else (NOT DISABLE_PS)
  message(STATUS "PS support disabled")
endif (NOT DISABLE_PS)

include_directories(
  SYSTEM ${EXTERN_INCLUDE_PATH}
)

link_directories(
  ${LIBRARY_OUTPUT_PATH}
  ${EXTERN_LIB_PATH}
)

include(ExternalProject)

add_subdirectory(minerva)
add_subdirectory(apps)
add_subdirectory(owl)
add_subdirectory(tests)
