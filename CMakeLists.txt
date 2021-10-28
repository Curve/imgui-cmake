cmake_minimum_required(VERSION 3.5)
project(imgui LANGUAGES CXX VERSION 1.85)

# --------------------------------------------------------------------------------------------------------
# Library options
# --------------------------------------------------------------------------------------------------------

option(imgui_demo "Include demo code" ON)
option(imgui_example "Build an example" OFF)
option(imgui_tables "Include tables code" ON)
option(imgui_widgets "Include widget code" ON)

set(imgui_backend_types None SDL GLFW GLUT WIN32)
set(imgui_render_types D3D9 D3D10 D3D11 D3D12 OpenGL2 OpenGL3 Vulkan WGPU SDL)

set(imgui_backend None CACHE STRING "The backend ImGui should use.")
set(imgui_render None CACHE STRING "The render-backend ImGui should use.")

set_property(CACHE imgui_render PROPERTY STRINGS ${imgui_render_types})
set_property(CACHE imgui_backend PROPERTY STRINGS ${imgui_backend_types})

if (NOT imgui_backend IN_LIST imgui_backend_types)
    message(FATAL_ERROR "ImGui backend must be one of ${imgui_backend_types}")
else()
    message(STATUS "Backend: ${imgui_backend}")
endif()
    
if (NOT imgui_render IN_LIST imgui_render_types)
    message(FATAL_ERROR "ImGui render-backend must be one of ${imgui_render_types}")
else()
    message(STATUS "Render-Backend: ${imgui_render}")
endif()

# --------------------------------------------------------------------------------------------------------
# Gather required files
# --------------------------------------------------------------------------------------------------------

file(GLOB src "imgui/imgui.cpp" "imgui/imgui_draw.cpp")

message(STATUS "Widgets: ${imgui_widgets}")
if (imgui_widgets)
    list(APPEND src "imgui/imgui_widgets.cpp")
endif()

message(STATUS "Demo: ${imgui_widgets}")
if (imgui_demo)
    list(APPEND src "imgui/imgui_demo.cpp")
endif()

message(STATUS "Tables: ${imgui_tables}")
if (imgui_tables)
    list(APPEND src "imgui/imgui_tables.cpp")
endif()

# --------------------------------------------------------------------------------------------------------
# Create the library
# --------------------------------------------------------------------------------------------------------

add_library(${PROJECT_NAME} STATIC ${src})
target_include_directories(${PROJECT_NAME} PUBLIC "imgui" "imgui/backends")

# --------------------------------------------------------------------------------------------------------
# Link the required renderer
# --------------------------------------------------------------------------------------------------------

if(${imgui_render} STREQUAL "Vulkan")
    if (NOT TARGET Vulkan::Vulkan)
        find_package(Vulkan REQUIRED)
    endif()
    target_link_libraries(${PROJECT_NAME} PUBLIC Vulkan::Vulkan)
    target_sources(${PROJECT_NAME} PRIVATE "imgui/backends/imgui_impl_vulkan.cpp")
elseif(${imgui_render} STREQUAL "OpenGL2")
    if(POLICY CMP0072)
        set(OpenGL_GL_PREFERENCE GLVND)
    endif()
    find_package(OpenGL "2" REQUIRED)
    target_link_libraries(${PROJECT_NAME} PUBLIC OpenGL::GL)
    target_sources(${PROJECT_NAME} PRIVATE "imgui/backends/imgui_impl_opengl2.cpp")
elseif(${imgui_render} STREQUAL "OpenGL3")
    if(POLICY CMP0072)
        set(OpenGL_GL_PREFERENCE GLVND)
    endif()
    find_package(OpenGL "3" REQUIRED)
    target_link_libraries(${PROJECT_NAME} PUBLIC OpenGL::GL)
    target_sources(${PROJECT_NAME} PRIVATE "imgui/backends/imgui_impl_opengl3.cpp")
    if (UNIX)
        target_link_libraries(${PROJECT_NAME} PRIVATE dl)
    endif()
elseif(${imgui_render} STREQUAL "WGPU")
    target_sources(${PROJECT_NAME} PRIVATE "imgui/backends/imgui_impl_wgpu.cpp")
elseif(${imgui_render} STREQUAL "SDL")
    target_sources(${PROJECT_NAME} PRIVATE "imgui/backends/imgui_impl_sdlrender.cpp")
elseif(${imgui_render} STREQUAL "D3D9")
    target_sources(${PROJECT_NAME} PRIVATE "imgui/backends/imgui_impl_dx9.cpp")
elseif(${imgui_render} STREQUAL "D3D10")
    target_sources(${PROJECT_NAME} PRIVATE "imgui/backends/imgui_impl_dx10.cpp")
elseif(${imgui_render} STREQUAL "D3D11")
    target_sources(${PROJECT_NAME} PRIVATE "imgui/backends/imgui_impl_dx11.cpp")
elseif(${imgui_render} STREQUAL "D3D12")
    target_sources(${PROJECT_NAME} PRIVATE "imgui/backends/imgui_impl_dx12.cpp")
endif()

# --------------------------------------------------------------------------------------------------------
# Link the required backend
# --------------------------------------------------------------------------------------------------------

if (${imgui_backend} STREQUAL "SDL")
    if (NOT TARGET SDL2::SDL2)
        find_package(SDL2 REQUIRED)
    endif()
    target_link_libraries(${PROJECT_NAME} PUBLIC SDL2::SDL2)
    target_sources(${PROJECT_NAME} PRIVATE "imgui/backends/imgui_impl_sdl.cpp")
elseif(${imgui_backend} STREQUAL "GLFW")
    if (NOT TARGET glfw)
        find_package(glfw3 REQUIRED)
    endif()
    target_link_libraries(${PROJECT_NAME} PUBLIC glfw)
    target_sources(${PROJECT_NAME} PRIVATE "imgui/backends/imgui_impl_glfw.cpp")
elseif(${imgui_backend} STREQUAL "GLUT")
    if (NOT TARGET GLUT::GLUT)
        find_package(GLUT REQUIRED)
    endif()
    target_link_libraries(${PROJECT_NAME} PUBLIC GLUT::GLUT)
    target_sources(${PROJECT_NAME} PRIVATE "imgui/backends/imgui_impl_glut.cpp")
elseif(${imgui_backend} STREQUAL "WIN32")
    target_sources(${PROJECT_NAME} PRIVATE "imgui/backends/imgui_impl_win32.cpp")
endif()

# --------------------------------------------------------------------------------------------------------
# Build example
# --------------------------------------------------------------------------------------------------------

if (imgui_example)
    message(STATUS "Building example")

    string(TOLOWER ${imgui_render} render_lower)
    string(TOLOWER ${imgui_backend} backend_lower)
    if (EXISTS "example_${backend_lower}_${render_lower}")
        add_executable(imgui_example "imgui/examples/example_${backend_lower}_${render_lower}/main.cpp")
        target_link_libraries(imgui_example PRIVATE imgui)
    else()
        message(FATAL_ERROR "Could not find example matching current configuration")
    endif()
endif()