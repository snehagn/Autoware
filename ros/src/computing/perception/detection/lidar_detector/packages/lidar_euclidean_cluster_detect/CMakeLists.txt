cmake_minimum_required(VERSION 2.8.3)
project(lidar_euclidean_cluster_detect)

find_package(catkin REQUIRED COMPONENTS
        pcl_ros
        roscpp
        geometry_msgs
        std_msgs
        sensor_msgs
        autoware_msgs
        tf
        jsk_recognition_msgs
        jsk_rviz_plugins
        vector_map_server
        grid_map_ros
        grid_map_cv
        grid_map_msgs
        )

find_package(OpenMP)
find_package(OpenCV REQUIRED)

set(CMAKE_CXX_FLAGS "-std=c++11 -O2 -Wall ${CMAKE_CXX_FLAGS}")

catkin_package(
        CATKIN_DEPENDS
        pcl_ros
        roscpp
        geometry_msgs
        std_msgs
        sensor_msgs
        autoware_msgs
        tf
        jsk_recognition_msgs
        jsk_rviz_plugins
        vector_map_server
        grid_map_ros
        grid_map_cv
        grid_map_msgs
        INCLUDE_DIRS include
)

include_directories(
        include
        ${catkin_INCLUDE_DIRS}
        ${OpenCV_INCLUDE_DIRS}
)
link_directories(${PCL_LIBRARY_DIRS})
link_directories(${OpenCV_LIBRARY_DIRS})

#Euclidean Cluster
add_executable(lidar_euclidean_cluster_detect
        nodes/lidar_euclidean_cluster_detect/lidar_euclidean_cluster_detect.cpp
        nodes/lidar_euclidean_cluster_detect/cluster.cpp)

find_package(CUDA)
if (${CUDA_FOUND})
    INCLUDE(FindCUDA)
    message("-- USING ACCELERATED CLUSTERING --")
    message("Version: " ${CUDA_VERSION})
    message("Library: " ${CUDA_CUDA_LIBRARY})
    message("Runtime: " ${CUDA_CUDART_LIBRARY})
    set(ROS_VERSION $ENV{ROS_DISTRO})
    if ("${ROS_VERSION}" MATCHES "(indigo)")
        set(CUDA_NVCC_FLAGS ${CUDA_NVCC_FLAGS};-std=c++11)
    endif()
    target_compile_definitions(lidar_euclidean_cluster_detect PRIVATE
            GPU_CLUSTERING=1
            )

    cuda_add_library(gpu_euclidean_clustering
            include/gpu_euclidean_clustering.h
            nodes/lidar_euclidean_cluster_detect/gpu_euclidean_clustering.cu
            )

    target_link_libraries(lidar_euclidean_cluster_detect
            ${OpenCV_LIBRARIES}
            ${catkin_LIBRARIES}
            ${PCL_LIBRARIES}
            gpu_euclidean_clustering)

else ()
    target_link_libraries(lidar_euclidean_cluster_detect
            ${OpenCV_LIBRARIES}
            ${catkin_LIBRARIES}
            ${PCL_LIBRARIES})

endif ()

add_dependencies(lidar_euclidean_cluster_detect
        ${catkin_EXPORTED_TARGETS}
        )

if (OPENMP_FOUND)
    set_target_properties(lidar_euclidean_cluster_detect PROPERTIES
            COMPILE_FLAGS ${OpenMP_CXX_FLAGS}
            LINK_FLAGS ${OpenMP_CXX_FLAGS}
            )
endif ()

install(TARGETS
        lidar_euclidean_cluster_detect
        ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
        LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
        RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION}
)