cmake_minimum_required(VERSION 3.10)
include(CMakePrintHelpers)
project(vulkan-play)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Define paths for external libraries
set(GLFW_ROOT "${CMAKE_SOURCE_DIR}/external/glfw")
set(GLFW_INCLUDE_DIR "${GLFW_ROOT}/include")
set(GLFW_LIBRARY_DIR "${GLFW_ROOT}/lib")

set(GLM_ROOT "${CMAKE_SOURCE_DIR}/external/glm")
set(GLM_INCLUDE_DIR "${GLM_ROOT}/include")

# Locate Vulkan SDK (First, try automatic detection)
find_package(Vulkan QUIET)

# If Vulkan is NOT found, manually set it using environment variables
if (NOT Vulkan_FOUND)
    if(DEFINED ENV{VULKAN_SDK})
        set(Vulkan_INCLUDE_DIR "$ENV{VULKAN_SDK}/macOS/include")
        set(Vulkan_LIBRARY "$ENV{VULKAN_SDK}/macOS/lib/libvulkan.dylib")
        message(STATUS "Manually setting Vulkan paths: ${Vulkan_INCLUDE_DIR}, ${Vulkan_LIBRARY}")
    else()
        message(FATAL_ERROR "Vulkan SDK not found. Please set the VULKAN_SDK environment variable.")
    endif()
endif()

# Include directories
include_directories(${GLFW_INCLUDE_DIR} ${Vulkan_INCLUDE_DIR} ${GLM_INCLUDE_DIR})

# Define source files
add_executable(${PROJECT_NAME} main.cpp)

# Lib directories
target_link_directories(${PROJECT_NAME} PRIVATE ${GLFW_LIBRARY_DIR})

# Platform-specific linking
if(WIN32)
    # Windows-specific Vulkan & GLFW linking
    target_link_libraries(${PROJECT_NAME} ${Vulkan_LIBRARIES} glfw3.lib opengl32.lib)
elseif(APPLE)
    # macOS-specific Vulkan & GLFW linking
    target_link_libraries(${PROJECT_NAME} ${Vulkan_LIBRARY} glfw3)
    cmake_print_variables(Vulkan_LIBRARY)
    # Ensure Vulkan's MoltenVK is linked properly
    target_link_libraries(${PROJECT_NAME} "-framework Cocoa")
    target_link_libraries(${PROJECT_NAME} "-framework QuartzCore")
    target_link_libraries(${PROJECT_NAME} "-framework Metal")
    target_link_libraries(${PROJECT_NAME} "-framework IOKit")
    target_link_libraries(${PROJECT_NAME} "-framework CoreFoundation")
endif()

# Set output directories (inside the local build folder)
set(LOCAL_BUILD_DIR "${CMAKE_SOURCE_DIR}/build")
set(OUTPUT_DIR "${LOCAL_BUILD_DIR}/${CMAKE_BUILD_TYPE}")

set_target_properties(${PROJECT_NAME} PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${OUTPUT_DIR}/bin"
    ARCHIVE_OUTPUT_DIRECTORY "${OUTPUT_DIR}/lib"
    LIBRARY_OUTPUT_DIRECTORY "${OUTPUT_DIR}/lib"
)

# On Windows, copy GLFW DLL to output directory
if (WIN32)
    add_custom_command(TARGET ${PROJECT_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
        ${GLFW_LIBRARY_DIR}/glfw3.dll
        "${OUTPUT_DIR}/bin"
    )
endif()