
<h1 align="center"> imgui-cmake </h1>
<p align="center">
A CMake compatible version of <a href="https://github.com/ocornut/imgui">imgui</a>
</p>

<p align="center">
    <a href="https://github.com/Curve/imgui-cmake/actions">
        <img src="https://img.shields.io/github/workflow/status/Curve/imgui-cmake/Test%20on%20Linux?label=Linux%20Build&style=flat-square" alt="Linux Build" />
    </a>
    <a href="https://github.com/Curve/imgui-cmake/actions">
        <img src="https://img.shields.io/github/workflow/status/Curve/imgui-cmake/Test%20on%20Windows?label=Windows%20Build&style=flat-square" alt="Windows Build" />
    </a>
    <a href="https://github.com/Curve/imgui-cmake/blob/master/LICENSE">
        <img src="https://img.shields.io/github/license/Curve/imgui-cmake.svg?style=flat-square" alt="License" />
    </a>
</p>

---

## ⚙️ Configuration
### Example
```cmake
set(imgui_example ON)
```
> Will build an example project
### Shared
```cmake
set(imgui_shared ON)
```
> Will build imgui as a shared library
### Demo
```cmake
set(imgui_demo ON)
```
> Will enable the demo code of ImGui
### Widgets
```cmake
set(imgui_widgets ON)
```
> Will enable ImGuis default widgets
### Tables
```cmake
set(imgui_tables ON)
```
> Will enable ImGuis table API
### Renderer
```cmake
set(imgui_render "<render-type>")
```
where <kbd>render-type</kbd> is one of:
* `D3D12`
* `D3D11`
* `D3D10`
* `D3D9`
* `OpenGL2`
* `OpenGL3`
* `Vulkan`
* `WGPU`
* `SDL`

### Backend
```cmake
set(imgui_backend "<backend-type>")
```
where <kbd>backend-type</kbd> is one of:
* `WIN32`
* `None`
* `GLFW`
* `GLUT`
* `SDL`

## 📎 Installation
- FetchContent
    ```cmake
    include(FetchContent)
    FetchContent_Declare(imgui GIT_REPOSITORY "https://github.com/Curve/imgui-cmake")

    FetchContent_MakeAvailable(imgui)
    target_link_libraries(<YourLibrary> imgui)
    ```
- Git Submodule
    ```bash
    git submodule add "https://github.com/Curve/imgui-cmake"
    ```
    ```cmake
    # Somewhere in your CMakeLists.txt
    add_subdirectory("<path_to_imgui>")
    target_link_libraries(<YourLibrary> imgui)
    ```

## 📓 Original Readme
The original project and readme can be found [here](https://github.com/ocornut/imgui#readme).