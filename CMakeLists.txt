cmake_minimum_required(VERSION 3.10)
project(vulkan-play)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Define paths
set(GLFW_ROOT "${CMAKE_SOURCE_DIR}/external/glfw")
set(GLFW_INCLUDE_DIR "${GLFW_ROOT}/include")
set(GLFW_LIBRARY_DIR "${GLFW_ROOT}/lib")

set(GLM_ROOT "${CMAKE_SOURCE_DIR}/external/glm")
set(GLM_INCLUDE_DIR "${GLM_ROOT}/include")

# Locate Vulkan
find_package(Vulkan REQUIRED)

# Include directories (GLFW, Vulkan, GLM)
include_directories(${GLFW_INCLUDE_DIR} ${Vulkan_INCLUDE_DIRS} ${GLM_INCLUDE_DIR})

# Define source files
add_executable(${PROJECT_NAME} main.cpp)

# Link Vulkan and GLFW
if (WIN32)
    target_link_directories(${PROJECT_NAME} PRIVATE ${GLFW_LIBRARY_DIR})
    target_link_libraries(${PROJECT_NAME} ${Vulkan_LIBRARIES} glfw3.lib opengl32.lib)
endif()

# Set output directories (inside the local build folder)
set(LOCAL_BUILD_DIR "${CMAKE_SOURCE_DIR}/build")
set_target_properties(${PROJECT_NAME} PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${LOCAL_BUILD_DIR}/bin"
    ARCHIVE_OUTPUT_DIRECTORY "${LOCAL_BUILD_DIR}/lib"
    LIBRARY_OUTPUT_DIRECTORY "${LOCAL_BUILD_DIR}/lib"
)

# Copy GLFW DLL to output directory (so the exe runs without missing DLLs)
if (WIN32)
    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
        ${GLFW_LIBRARY_DIR}/glfw3.dll
        ${LOCAL_BUILD_DIR}/bin
    )
endif()
