# bgfx_cmake

This repo contains cmake configuration files that can be used to build bgfx with CMake.

## Building

```bash
git submodule add -b master https://github.com/DendyA/bgfx.cmake.git {folder_to_store}
cd {folder_to_store}
git submodule init
git submodule update
```
The top command will download the bgfx_cmake project as a submodule to the root project. The last two commands will init and pull in the tracked commit in the bgfx, bimg, and bx submodules in this submodule.

Once initialized and downloaded, this project (and the subsequent bgfx library and its dependencies) can be included into the root project by adding the following two lines to the root project's CMakeLists.txt.

```cmake
ADD_SUBDIRECTORY({path_from_root_project's_CMakeLists.txt_to_cmake_bgfx_folder})

TARGET_LINK_LIBRARIES({TARGET_NAME} {PUBLIC|PRIVATE|INTERFACE}
        bgfx
)
```

One note is that the ```TARGET_LINK_LIBRARIES``` needs to be added after either a call to ```ADD_LIBRARY``` or ```ADD_EXECUTABLE``` in the root project's CMakeLists.txt. And ```ADD_SUBDIRECTORY``` needs to be added before ```ADD_LIBRARY``` or ```ADD_EXECUTABLE```.

Another note is that in the code example above, the {}s need to be replaced with your project specific information.

## Features
* No outside dependencies besides bx, bimg, bgfx, and CMake.
* Tested on
    * Linux (Ubuntu 22.04)
* Compiles bgfx, tools & examples.
* Detects shader modifications and automatically rebuilds them for all examples.

## Added cmake commands
bgfx.cmake will install `bgfxToolUtils.cmake` which has useful cmake functions for using bgfx's tools:

### `bgfx_compile_binary_to_header`
Add a build rule for a binary file to the generated build system using bin2c.
```cmake
bgfx_compile_binary_to_header(
	INPUT_FILE filename
	OUTPUT_FILE filename
	ARRAY_NAME name
)
```
This defines a bin2c command to generate a specified `OUTPUT_FILE` header with an array `ARRAY_NAME` with the binary representation of a `INPUT_FILE` file.

Adding these `INPUT_FILE` as source files to a target will run `bin2c` at build time and they will rebuild if either the contents of the `INPUT_FILE` change.

#### Examples: Generating an image as a header
```cmake
bgfx_compile_binary_to_header(
  INPUT_FILE image.png
  OUTPUT_FILE ${CMAKE_BINARY_DIR}/include/generated/images/image.png.h
)
add_library(myLib image.png)
target_include_directories(myLib ${CMAKE_BINARY_DIR}/include/generated/images)
```

```cpp
// main.cpp
#include <image.png.h>
```

### `bgfx_compile_texture`
Add a build rule for a texture to the generated build system be compiled using texturec.
```cmake
bgfx_compile_texture(
     FILE filename
     OUTPUT filename
     [FORMAT format]
     [QUALITY default|fastest|highest]
     [MIPS]
     [MIPSKIP N]
     [NORMALMAP]
     [EQUIRECT]
     [STRIP]
     [SDF]
     [REF alpha]
     [IQA]
     [PMA]
     [LINEAR]
     [MAX max size]
     [RADIANCE model]
     [AS extension]
)
```

### `bgfx_compile_shader_to_header`
Add a build rule for a `*.sc` shader to the generated build system using shaderc.
```cmake
bgfx_compile_shader_to_header(
	TYPE VERTEX|FRAGMENT|COMPUTE
	SHADERS filenames
	VARYING_DEF filename
	OUTPUT_DIR directory
)
```
This defines a shaderc command to generate headers for a number of `TYPE` shaders with `SHADERS` files and `VARYING_DEF` file in the `OUTPUT_DIR` directory. There will be one generated shader for each supported rendering API on this current platform according to the `BGFX_EMBEDDED_SHADER` macro in `bgfx/embedded_shader.h`.

The generated headers will have names in the format of `${SHADERS}.${RENDERING_API}.bin.h` where `RENDERING_API` can be `glsl`, `essl`, `spv`, `dx9`, `dx11` and `mtl` depending on the availability of the platform.

Adding these `SHADERS` as source files to a target will run `shaderc` at build time and they will rebuild if either the contents of the `SHADERS` or the `VARYING_DEF` change.

#### Examples: Generating shaders as headers
```cmake
bgfx_compile_shader_to_header(
  TYPE VERTEX
  SHADERS vs.sc
  VARYING_DEF varying.def.sc
  OUTPUT_DIR ${CMAKE_BINARY_DIR}/include/generated/shaders
)
bgfx_compile_shader_to_header(
  TYPE FRAGMENT
  SHADERS fs.sc
  VARYING_DEF ${CMAKE_SOURCE_DIR}/varying.def.sc
  OUTPUT_DIR ${CMAKE_BINARY_DIR}/include/generated/shaders
)

add_library(myLib main.cpp vs.sc fs.sc)
target_include_directories(myLib ${CMAKE_BINARY_DIR}/include/generated/shaders)
```

```cpp
// main.cpp
#include <vs.sc.glsl.bin.h>
#include <vs.sc.essl.bin.h>
#include <vs.sc.spv.bin.h>
#include <fs.sc.glsl.bin.h>
#include <fs.sc.essl.bin.h>
#include <fs.sc.spv.bin.h>
#if defined(_WIN32)
#include <vs.sc.dx9.bin.h>
#include <vs.sc.dx11.bin.h>
#include <fs.sc.dx9.bin.h>
#include <fs.sc.dx11.bin.h>
#endif //  defined(_WIN32)
#if __APPLE__
#include <vs.sc.mtl.bin.h>
#include <fs.sc.mtl.bin.h>
#endif // __APPLE__

const bgfx::EmbeddedShader k_vs = BGFX_EMBEDDED_SHADER(vs);
const bgfx::EmbeddedShader k_fs = BGFX_EMBEDDED_SHADER(fs);
```
