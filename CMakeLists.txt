cmake_minimum_required(VERSION 3.16)
project(imgui LANGUAGES CXX)

# --------------------------------------------------------------------------------------------------------
# Library options
# --------------------------------------------------------------------------------------------------------

option(imgui_demo         "Include demo code (ShowDemoWindow)"          ON)
option(imgui_shared       "Build ImGui as a shared library"            OFF)
option(imgui_tables       "Include ImGuis tables library"               ON)
option(imgui_widgets      "Include ImGuis widget library"               ON)

# --------------------------------------------------------------------------------------------------------
# Library config
# --------------------------------------------------------------------------------------------------------

set(imgui_version  "v1.89.8" CACHE STRING "The ImGui version to use")
set(imgui_renderer "Unknown" CACHE STRING "The ImGui renderer to use")
set(imgui_platform "Unknown" CACHE STRING "The ImGui platform (backend) to use")

# --------------------------------------------------------------------------------------------------------
# Ensure valid config
# --------------------------------------------------------------------------------------------------------

set(supported_renderers "DirectX9" "DirectX10" "DirectX11" "DirectX12" "Metal" "OpenGL2" "OpenGL3" "SDL_Renderer2" "SDL_Renderer3" "Vulkan" "WebGPU")
set(supported_platforms "Unknown" "GLFW" "SDL2" "SDL3" "Win32" "Glut" "OSX" "Android")

set_property(CACHE imgui_renderer PROPERTY STRINGS ${supported_renderers})
set_property(CACHE imgui_platform PROPERTY STRINGS ${supported_platforms})

if (NOT imgui_renderer IN_LIST supported_renderers)
    message(FATAL_ERROR "Renderer must be one of ${supported_renderers}")
endif()

if (NOT imgui_platform IN_LIST supported_platforms)
    message(FATAL_ERROR "Renderer must be one of ${supported_platforms}")
endif()

# --------------------------------------------------------------------------------------------------------
# Conditionally Shared / Static
# --------------------------------------------------------------------------------------------------------

set(TYPE STATIC)

if (imgui_shared)
    set(TYPE SHARED)
endif()

# --------------------------------------------------------------------------------------------------------
# Setup library
# --------------------------------------------------------------------------------------------------------

add_library(${PROJECT_NAME} ${TYPE})
add_library(ocornut::${PROJECT_NAME} ALIAS ${PROJECT_NAME})

# --------------------------------------------------------------------------------------------------------
# Obtain ImGui Source
# --------------------------------------------------------------------------------------------------------

include("cmake/cpm.cmake")

CPMAddPackage(
  NAME               imgui
  GIT_TAG            ${imgui_version}
  GITHUB_REPOSITORY  "ocornut/imgui"
  DOWNLOAD_ONLY
)

# --------------------------------------------------------------------------------------------------------
# Setup includes
# --------------------------------------------------------------------------------------------------------

target_include_directories(${PROJECT_NAME} PUBLIC "${imgui_SOURCE_DIR}")
target_include_directories(${PROJECT_NAME} PUBLIC "${imgui_SOURCE_DIR}/backends")

# --------------------------------------------------------------------------------------------------------
# Setup sources
# --------------------------------------------------------------------------------------------------------

target_sources(${PROJECT_NAME} PRIVATE 
    "${imgui_SOURCE_DIR}/imgui.cpp"
    "${imgui_SOURCE_DIR}/imgui_draw.cpp"
)

if (imgui_tables)
    target_sources(${PROJECT_NAME} PRIVATE "${imgui_SOURCE_DIR}/imgui_tables.cpp")
endif()

if (imgui_widgets)
    target_sources(${PROJECT_NAME} PRIVATE "${imgui_SOURCE_DIR}/imgui_widgets.cpp")
endif()

if (imgui_demo)
    target_sources(${PROJECT_NAME} PRIVATE "${imgui_SOURCE_DIR}/imgui_demo.cpp")
endif()

# --------------------------------------------------------------------------------------------------------
# Setup Renderers
# --------------------------------------------------------------------------------------------------------

if (imgui_renderer STREQUAL "DirectX9")
    target_link_libraries(${PROJECT_NAME} PRIVATE d3d9)
    target_sources(${PROJECT_NAME} PRIVATE "${imgui_SOURCE_DIR}/backends/imgui_impl_dx9.cpp")
endif()

if (imgui_renderer STREQUAL "DirectX10")
    target_link_libraries(${PROJECT_NAME} PRIVATE d3d10)
    target_sources(${PROJECT_NAME} PRIVATE "${imgui_SOURCE_DIR}/backends/imgui_impl_dx10.cpp")
endif()

if (imgui_renderer STREQUAL "DirectX11")
    target_link_libraries(${PROJECT_NAME} PRIVATE d3d11)
    target_sources(${PROJECT_NAME} PRIVATE "${imgui_SOURCE_DIR}/backends/imgui_impl_dx11.cpp")
endif()

if (imgui_renderer STREQUAL "DirectX12")
    target_link_libraries(${PROJECT_NAME} PRIVATE d3d12)
    target_sources(${PROJECT_NAME} PRIVATE "${imgui_SOURCE_DIR}/backends/imgui_impl_dx12.cpp")
endif()

if (imgui_renderer STREQUAL "Metal")
    # This might need additional libraries linked, however I don't have a mac to test this with.
    target_sources(${PROJECT_NAME} PRIVATE "${imgui_SOURCE_DIR}/backends/imgui_impl_metal.mm")
endif()

if (imgui_renderer STREQUAL "OpenGL2")
    find_package(OpenGL REQUIRED)
    target_link_libraries(${PROJECT_NAME} PUBLIC OpenGL::GL)
    target_sources(${PROJECT_NAME} PRIVATE "${imgui_SOURCE_DIR}/backends/imgui_impl_opengl2.cpp")
endif()

if (imgui_renderer STREQUAL "OpenGL3")
    find_package(OpenGL REQUIRED)
    target_link_libraries(${PROJECT_NAME} PUBLIC OpenGL::GL)
    target_sources(${PROJECT_NAME} PRIVATE "${imgui_SOURCE_DIR}/backends/imgui_impl_opengl3.cpp")
endif()

if (imgui_renderer STREQUAL "SDL_Renderer2")
    find_package(SDL2 REQUIRED)
    target_link_libraries(${PROJECT_NAME} PUBLIC SDL2::SDL2)
    target_sources(${PROJECT_NAME} PRIVATE "${imgui_SOURCE_DIR}/backends/imgui_impl_sdlrenderer2.cpp")
endif()

if (imgui_renderer STREQUAL "SDL_Renderer3")
    find_package(SDL3 REQUIRED)
    target_link_libraries(${PROJECT_NAME} PUBLIC SDL3::SDL3)
    target_sources(${PROJECT_NAME} PRIVATE "${imgui_SOURCE_DIR}/backends/imgui_impl_sdlrenderer3.cpp")
endif()

if (imgui_renderer STREQUAL "Vulkan")
    find_package(Vulkan REQUIRED)
    target_link_libraries(${PROJECT_NAME} PUBLIC Vulkan::Vulkan)
    target_sources(${PROJECT_NAME} PRIVATE "${imgui_SOURCE_DIR}/backends/imgui_impl_vulkan.cpp")
endif()

if (imgui_renderer STREQUAL "WebGPU")
    target_sources(${PROJECT_NAME} PRIVATE "${imgui_SOURCE_DIR}/backends/imgui_impl_wgpu.cpp")
endif()

# --------------------------------------------------------------------------------------------------------
# Setup Platforms
# --------------------------------------------------------------------------------------------------------

if (imgui_platform STREQUAL "GLFW")
    find_package(glfw3 REQUIRED)
    target_link_libraries(${PROJECT_NAME} PUBLIC glfw)
    target_sources(${PROJECT_NAME} PRIVATE "${imgui_SOURCE_DIR}/backends/imgui_impl_glfw.cpp")
endif()

if (imgui_platform STREQUAL "SDL2")
    find_package(SDL2 REQUIRED)
    target_link_libraries(${PROJECT_NAME} PUBLIC SDL2::SDL2)
    target_sources(${PROJECT_NAME} PRIVATE "${imgui_SOURCE_DIR}/backends/imgui_impl_sdl2.cpp")
endif()

if (imgui_platform STREQUAL "SDL3")
    find_package(SDL3 REQUIRED)
    target_link_libraries(${PROJECT_NAME} PUBLIC SDL3::SDL3)
    target_sources(${PROJECT_NAME} PRIVATE "${imgui_SOURCE_DIR}/backends/imgui_impl_sdl3.cpp")
endif()

if (imgui_platform STREQUAL "Win32")
    target_sources(${PROJECT_NAME} PRIVATE "${imgui_SOURCE_DIR}/backends/imgui_impl_win32.cpp")
endif()

if (imgui_platform STREQUAL "Glut")
    find_package(GLUT REQUIRED)
    target_link_libraries(${PROJECT_NAME} PUBLIC GLUT::GLUT)
    target_sources(${PROJECT_NAME} PRIVATE "${imgui_SOURCE_DIR}/backends/imgui_impl_glut.cpp")
endif()

if (imgui_platform STREQUAL "OSX")
    target_sources(${PROJECT_NAME} PRIVATE "${imgui_SOURCE_DIR}/backends/imgui_impl_osx.mm")
endif()

if (imgui_platform STREQUAL "Android")
    target_sources(${PROJECT_NAME} PRIVATE "${imgui_SOURCE_DIR}/backends/imgui_impl_android.cpp")
endif()