cmake_minimum_required(VERSION 3.16)

set(CMAKE_CXX_STANDARD 17)

project(Barycentric)
configure_file(${PROJECT_SOURCE_DIR}/configuration/root_directory.h.in configuration/root_directory.h)
include_directories(${CMAKE_BINARY_DIR}/configuration)
include_directories(${PROJECT_SOURCE_DIR}/includes)

set(GLFW_DIR "C:/Projects/Studies_in_C/Exercises/openGL")
include_directories(${GLFW_DIR}/include)
link_directories(${GLFW_DIR}/lib)

add_executable(Barycentric stb_image/stb_image.cpp Shader.h glad.c Barycentric.cpp)

add_custom_target(copy_shaders ALL
    COMMAND ${CMAKE_COMMAND} -E copy_directory
    ${PROJECT_SOURCE_DIR}/Shaders $<TARGET_FILE_DIR:Barycentric>/Shaders
)
add_custom_command(TARGET Barycentric POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy_directory
    ${PROJECT_SOURCE_DIR}/Shaders $<TARGET_FILE_DIR:Barycentric>/Shaders
)

target_link_libraries(Barycentric glfw3dll opengl32 gdi32)
