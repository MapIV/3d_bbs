cmake_minimum_required(VERSION 3.10)
project(ros_bbs3d)

find_package(ament_cmake_auto REQUIRED)
ament_auto_find_build_dependencies()

# Default to C++17
if (NOT CMAKE_CXX_STANDARD)
  set(CMAKE_CXX_STANDARD 17)
endif ()

if (CMAKE_COMPILER_IS_GNUCXX OR CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  add_compile_options(-Wall -Wextra -Wpedantic -Wunused-function)
endif ()

find_package(ament_cmake_auto REQUIRED)
find_package(Eigen3 REQUIRED)
find_package(CUDA REQUIRED)
find_package(OpenMP)
if (OPENMP_FOUND)
  set (CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${OpenMP_C_FLAGS}")
  set (CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${OpenMP_CXX_FLAGS}")
endif()

include_directories(
  include
  ${EIGEN3_INCLUDE_DIR}
  ${CUDA_INCLUDE_DIRS}
)

link_directories(${CUDA_LIBRARY_DIRS})

ament_auto_add_library(cpu_bbs3d
  SHARED
  src/cpu_bbs3d/bbs3d.cpp
  src/cpu_bbs3d/voxelmaps.cpp
  src/cpu_bbs3d/voxelmaps_io.cpp
)

cuda_add_library(gpu_bbs3d
  SHARED
  src/gpu_bbs3d/bbs3d.cu
  src/gpu_bbs3d/calc_score.cu
  src/gpu_bbs3d/voxelmaps.cu
  src/gpu_bbs3d/stream_manager/check_error.cu
  src/gpu_bbs3d/voxelmaps_io.cu
)
install(
  TARGETS gpu_bbs3d
  EXPORT export_gpu_bbs3d
  ARCHIVE DESTINATION lib
  LIBRARY DESTINATION lib
  RUNTIME DESTINATION bin
)

install(
  DIRECTORY include/
  DESTINATION include/${PROJECT_NAME}
)

ament_export_include_directories("include/${PROJECT_NAME}")
ament_export_libraries(gpu_bbs3d)
ament_export_targets(export_gpu_bbs3d)

ament_auto_package()

#install(TARGETS cpu_bbs3d DESTINATION lib)
#install(TARGETS gpu_bbs3d DESTINATION lib)
#install(DIRECTORY bbs3d/include/ DESTINATION include)