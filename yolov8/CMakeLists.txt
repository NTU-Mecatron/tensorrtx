cmake_minimum_required(VERSION 3.10)

project(yolov8)

add_definitions(-std=c++11)
add_definitions(-DAPI_EXPORTS)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_BUILD_TYPE Debug)

# Suppress warnings
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -w")
set(CMAKE_CUDA_FLAGS "${CMAKE_CUDA_FLAGS} -w")

set(CMAKE_CUDA_COMPILER /usr/local/cuda-12.1/bin/nvcc)
enable_language(CUDA)

# Use ccache for C++ and CUDA compilers
find_program(CCACHE_PROGRAM ccache)
if(CCACHE_PROGRAM)
    set(CMAKE_CXX_COMPILER_LAUNCHER ${CCACHE_PROGRAM})
    set(CMAKE_CUDA_COMPILER_LAUNCHER ${CCACHE_PROGRAM})
endif()

# Include CUDA directories
include_directories(/usr/local/cuda-12.1/include)
link_directories(/usr/local/cuda-12.1/lib64)

include_directories(${PROJECT_SOURCE_DIR}/include)
include_directories(${PROJECT_SOURCE_DIR}/plugin)

# include and link dirs of cuda and tensorrt, you need adapt them if yours are different
if (CMAKE_SYSTEM_PROCESSOR MATCHES "aarch64")
  message("embed_platform on")
  include_directories(/usr/local/cuda/targets/aarch64-linux/include)
  link_directories(/usr/local/cuda/targets/aarch64-linux/lib)
else()
  message("embed_platform off")
  # cuda
  include_directories(/usr/local/cuda/include)
  link_directories(/usr/local/cuda/lib64)

  # tensorrt
  include_directories(/usr/include/x86_64-linux-gnu/)
  link_directories(/usr/lib/x86_64-linux-gnu/)

endif()

add_library(myplugins SHARED ${PROJECT_SOURCE_DIR}/plugin/yololayer.cu)
target_link_libraries(myplugins nvinfer cudart)

find_package(OpenCV)
include_directories(${OpenCV_INCLUDE_DIRS})


file(GLOB_RECURSE SRCS ${PROJECT_SOURCE_DIR}/src/*.cpp ${PROJECT_SOURCE_DIR}/src/*.cu)

add_executable(yolov8_det ${PROJECT_SOURCE_DIR}/yolov8_det.cpp ${SRCS})
target_link_libraries(yolov8_det nvinfer cudart myplugins ${OpenCV_LIBS})

# add_executable(yolov8_seg ${PROJECT_SOURCE_DIR}/yolov8_seg.cpp ${SRCS})
# target_link_libraries(yolov8_seg nvinfer cudart myplugins ${OpenCV_LIBS})


# add_executable(yolov8_pose ${PROJECT_SOURCE_DIR}/yolov8_pose.cpp ${SRCS})
# target_link_libraries(yolov8_pose nvinfer cudart myplugins ${OpenCV_LIBS})

# add_executable(yolov8_cls ${PROJECT_SOURCE_DIR}/yolov8_cls.cpp ${SRCS})
# target_link_libraries(yolov8_cls nvinfer cudart myplugins ${OpenCV_LIBS})
