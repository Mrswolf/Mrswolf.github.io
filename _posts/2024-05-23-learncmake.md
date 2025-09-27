---
title: CMake Cheatsheet
categories: [c++]
permalink: /learncmake/
date: 2024-05-23 00:00:00
updated: 2024-06-24 00:00:00
tags: [c++]
toc: true
---
<!-- toc -->

CMake is a popular cross-platform build system that allows developers to use a common interface to define a set of rules to build the source code with different compilers, such as GCC, Clang and Visual Studio. In fact, CMake supports not only C/C++, but also other languages such as C# and Fortran. CMake is an essential tool for building software projects and making the whole process easier for anyone. <!--more-->

I put my code [here](https://github.com/Mrswolf/blog_code/tree/main/cmake).

## Basic
### How to start
After you install CMake, the starting point is to create a `CMakeLists.txt` file. **Three** basic commands are required for the minimum build process. Below is a basic `CMakeLists.txt` file template:
```cmake
cmake_minimum_required(VERSION 3.12)
project(HelloWorld VERSION 1.0 LANGUAGES C CXX)
set(CMAKE_CXX_STANDARD 11)
add_executable(helloworld main.cpp)
```

Don't forget to create a source file named `main.cpp` alongside with the `CMakeLists.txt` file:
```cpp
#include <iostream>

int main(int argc, char *argv[])
{
    std::cout << "HELLO CMAKE!" << std::endl;
    return 0;
}
```

To build the project, open the terminal where you create your `CMakeList.txt` file, create a new folder `mkdir build && cd build` and just type `cmake ..`. CMake would output a bunch of files for building the system, the last step is to actually compile&link the project.  Depending on the underlying generators, we could build the project with `make` or `ninja` or a more general command `cmake --build . --config Release`. The default behavior is to generate an executable object with release configs on the current folder. Type `./helloworld` to run the program.

CMake uses the jargon **target** refering to those final executable objects or libraries, e.g. `helloworld` in this demo. `LANGUAGES C CXX` specifies the programming languages needed to build the project. `CMAKE_CXX_STANDARD` specifies the c++ standard to build the project.

### How to find headers
So far we have only compiled one source file, but what if we have hundreds of source files and header files scattered across multiple directories? I created a folder structure looks like this:
```
includes/
  algo.h
algo.cpp
main.cpp
CMakeLists.txt
```
To compile this project, we tell CMake that we also need to compile `algo.cpp` and `algo.h`:
```cmake
cmake_minimum_required(VERSION 3.12)
project(HelloWorld VERSION 1.0 LANGUAGES C CXX)
set(CMAKE_CXX_STANDARD 11)
add_executable(helloworld main.cpp algo.cpp)
target_include_directories(helloworld PRIVATE includes)
```

`target_include_directories` refers to where to seach for additional headers, the `includes` directory in this case. And `PRIVATE` refers to that these header files are only visible for the target `helloworld`, not for any target linking against the target defined here (if `helloworld` is a library). `include_directories` acts as the same purpose but for all targets in the current CMakeLists.

### How to collect all sources
Instead of manual settings, an easy way to get all cpp files is to use `file` command:
```cmake
cmake_minimum_required(VERSION 3.12)
project(HelloWorld VERSION 1.0 LANGUAGES C CXX)
set(CMAKE_CXX_STANDARD 11)
file(GLOB SOURCES *.cpp utils/*.cpp)
add_executable(helloworld ${SOURCES})
target_include_directories(helloworld PRIVATE includes)
```
which collects files under the current and `utils` directories with the postfix `.cpp` and stores into the variable `SOURCES`.

Another way to collect all source files is to use command `aux_source_directory` like this:
```cmake
cmake_minimum_required(VERSION 3.12)
project(HelloWorld VERSION 1.0 LANGUAGES C CXX)
set(CMAKE_CXX_STANDARD 11)
aux_source_directory(.  utils SOURCES)
add_executable(helloworld ${SOURCES})
target_include_directories(helloworld PRIVATE includes)
```
### How to add definitions
It's normal to add preprocessor definitions to control the compiling process. Before version 3.12, CMake used command `add_definitions` which affects all targets in the current directory and subdirectories, even if you don't want to. In version 3.12, CMake introduced a new command `add_compile_definitions` which is more specific to targets in the current CMakeLists. It's generally recommended to move towards to the new command.

The following code defines `EXPORT_DLL`:
```cmake
add_definitions(-DEXPORT_DLL)
add_compile_definitions(EXPORT_DLL)
```

It's also possible to add definitions to a specific target:
```cmake
target_compile_definitions(helloworld PRIVATE EXPORT_DLL)
```

### How to pass compiler flags
Sometimes we want to control the compiler behaviors more precisely. CMake has `add_compile_options` for all targets in the current directory and subdirectories, and `targe_compile_options` for a specific target.

The following code tells gcc/g++ to use `O3` optimization level and generate debug information to be used by GDB:
```cmake
add_compile_options(-O3)
target_compile_options(helloworld PRIVATE -g)
```

### How to designate output folders
There are some predefined variables to specifies the output directories:
- `CMAKE_RUNTIME_OUTPUT_DIRECTORY` refers to where to put executable files (`.exe`) created by `add_executable` or `.dll` created by `add_library` with `SHARED` on Windows platform
- `CMAKE_LIBRARY_OUTPUT_DIRECTORY` refers to where to put shared libraries (`.so`) created by `add_library` with `SHARED` option on Linux platform
- `CMAKE_ARCHIVE_OUTPUT_DIRECTORY` refers to where to put static libraries (`.a`, `.lib`) created by `add_library` with `STATIC` option or `.lib` created by `add_library` with `SHARED` option on Windows platform

CMake also has a few useful variables to specify directories:
- `CMAKE_SOURCE_DIR` contains the directory where the called `CMakeLists` exists
- `CMAKE_BINARY_DIR` contains the directory where you generate those temporary files to build the project

For our helloworld project, the executable `helloworld` would be put under the `lib` folder:
```cmake
cmake_minimum_required(VERSION 3.12)
project(HelloWorld VERSION 1.0 LANGUAGES C CXX)
set(CMAKE_CXX_STANDARD 11)
file(GLOB SOURCES "*.cpp" "utils/*.cpp")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib)
add_executable(helloworld ${SOURCES})
target_include_directories(helloworld PRIVATE includes)
```

### How to compile static libraries
Just use `add_library` instead of `add_executable`:
```cmake
cmake_minimum_required(VERSION 3.12)
project(stalib VERSION 1.0 LANGUAGES C CXX)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib)
aux_source_directory(. SOURCES)
add_library(stalib STATIC ${SOURCES})
```
This CMakeLists builds a static library `libstalib.a`. 

### How to compile shared libraries
I also made a new folder `dynlib` under my `helloword` project. CMake compiles shared libraries by `add_library` command with `SHARED` option:
```cmake
cmake_minimum_required(VERSION 3.12)
project(dynlib VERSION 1.0 LANGUAGES C CXX)
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib)
aux_source_directory(. SOURCES)
add_library(dynlib SHARED ${SOURCES})
target_compile_definitions(dynlib PRIVATE EXPORT_DLL)
target_compile_options(dynlib PRIVATE -fvisibility=hidden -fvisibility-inlines-hidden)
```
The above config generates `libdynlib.so`, which can be linked into a larger program.

By default, Linux exports all symbols when compiles a shared library. However, Windows does the opposite thing. To control the visibility of symbols on Linux, we could add `-fvisibility=hidden -fvisibility-inlines-hidden` to the compiler to make sure all symbols unvisible, until we make it explicitly by adding `__attribute__((visibility('default')))` before symbols (Windows uses `__declspec(dllexport)` and `__declspec(dllimport)`). This would greatly reduce the chance of symbol collision. For compatible code, we could use [macro definitions](https://gcc.gnu.org/wiki/Visibility):
```cpp
#if defined _WIN32 || defined __CYGWIN__
  #define HELPER_DLL_IMPORT __declspec(dllimport)
  #define HELPER_DLL_EXPORT __declspec(dllexport)
  #define HELPER_DLL_LOCAL
#else
  #if __GNUC__ >= 4
    #define HELPER_DLL_IMPORT __attribute__ ((visibility ("default")))
    #define HELPER_DLL_EXPORT __attribute__ ((visibility ("default")))
    #define HELPER_DLL_LOCAL  __attribute__ ((visibility ("hidden")))
  #else
    #define HELPER_DLL_IMPORT
    #define HELPER_DLL_EXPORT
    #define HELPER_DLL_LOCAL
  #endif
#endif

#ifdef EXPORT_ALL
  #define DLL_API HELPER_DLL_EXPORT
#else
  #define DLL_API HELPER_DLL_IMPORT
#endif

DLL_API void hello();

void hello(int i);
```

Use command `nm -C -D libdynlib.so` to list all exported symbols:
```
                 w _ITM_deregisterTMCloneTable
                 w _ITM_registerTMCloneTable
0000000000001199 T hello()
                 U std::ostream::operator<<(std::ostream& (*)(std::ostream&))@GLIBCXX_3.4
                 U std::ostream::operator<<(int)@GLIBCXX_3.4
```
we can see that `void hello(int i)` is not exported.

### How to link libraries
Use `target_link_libraries` to link static and shared libraries for a single target:
```cmake
cmake_minimum_required(VERSION 3.12)
project(HelloWorld VERSION 1.0 LANGUAGES C CXX)
set(CMAKE_CXX_STANDARD 11)
file(GLOB SOURCES "*.cpp" "utils/*.cpp")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib)
add_executable(helloworld ${SOURCES})
target_include_directories(helloworld PRIVATE includes)
target_link_libraries(helloworld PRIVATE stalib dynlib)
```
or `link_libraries` which affects all targets created later in the current directory and subdirectories.

### How to add library seaching paths
It's also common to add library seaching paths. CMake uses `link_directories` and `target_link_directories` (available in version 3.13) to add library paths, for targets created later and a single target, respectively.
```cmake
cmake_minimum_required(VERSION 3.12)
project(HelloWorld VERSION 1.0 LANGUAGES C CXX)
set(CMAKE_CXX_STANDARD 11)
file(GLOB SOURCES "*.cpp" "utils/*.cpp")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib)
add_executable(helloworld ${SOURCES})
target_include_directories(helloworld PRIVATE includes)
target_link_directories(helloworld PRIVATE ${CMAKE_SOURCE_DIR}/lib)
target_link_libraries(helloworld PRIVATE stalib dynlib)
```
These library searching paths can be found in a preset variable `CMAKE_LIBRARY_PATH`.

### How to compile multiple targets on a single CMakeLists
`add_subdirectory` searches available CMakeLists on the subdirectory so that it can be compiled on the top-level called CMakeLists:
```cmake
cmake_minimum_required(VERSION 3.13)
project(HelloWorld VERSION 1.0 LANGUAGES C CXX)

add_subdirectory(dynlib)
add_subdirectory(stalib)

set(CMAKE_CXX_STANDARD 11)
file(GLOB SOURCES "*.cpp" "utils/*.cpp")
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib)
add_executable(helloworld ${SOURCES})
target_include_directories(helloworld PRIVATE includes)
target_link_directories(helloworld PRIVATE ${CMAKE_SOURCE_DIR}/lib)
target_link_libraries(helloworld PRIVATE stalib dynlib)
```
This CMakeLists firstly builds two libraries and then build the executable object `helloworld`, which links these libraries.

### How to compile a CUDA program
CMake automatically triggers `nvcc` for compiling files with the `.cu` extension. I would compile a cuda kernel funtion into `dynlib` and call it in the main program. To do that, I created a new `dyncuda.cu` file:
```cpp
#include "dynlib.h"
#include <iostream>

#define CHECK_CUDA_ERROR(val) check((val), #val, __FILE__, __LINE__)
void check(cudaError_t err, const char *const func, const char *const file,
           const int line)
{
    if (err != cudaSuccess)
    {
        std::cerr << "CUDA Runtime Error at: " << file << ":" << line
                  << std::endl;
        std::cerr << cudaGetErrorString(err) << " " << func << std::endl;
        // We don't exit when we encounter CUDA errors in this example.
        // std::exit(EXIT_FAILURE);
    }
}

#define CHECK_LAST_CUDA_ERROR() checkLast(__FILE__, __LINE__)
void checkLast(const char *const file, const int line)
{
    cudaError_t const err{cudaGetLastError()};
    if (err != cudaSuccess)
    {
        std::cerr << "CUDA Runtime Error at: " << file << ":" << line
                  << std::endl;
        std::cerr << cudaGetErrorString(err) << std::endl;
        // We don't exit when we encounter CUDA errors in this example.
        // std::exit(EXIT_FAILURE);
    }
}

__global__ void cudahello()
{
    printf("Hello cuda\n");
}

void cudaHelloLaunch()
{
    cudahello<<<1, 1>>>();
    CHECK_CUDA_ERROR(cudaDeviceSynchronize());
    CHECK_LAST_CUDA_ERROR();
}
```
and its header looks like:
```cpp
__global__ void cudahello();

DLL_API void cudaHelloLaunch();
```

In this way, the cuda kernel function would be launched by a cpp wrapper function exported to other programs. CMakeLists compiling the CUDA program looks like this:
```cmake
cmake_minimum_required(VERSION 3.13)
project(dynlib VERSION 1.0 LANGUAGES C CXX CUDA)
set(CMAKE_CXX_STANDARD 11)

if(NOT DEFINED CMAKE_CUDA_ARCHITECTURES)
  set(CMAKE_CUDA_ARCHITECTURES 61;70)
endif()

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_SOURCE_DIR}/lib)

aux_source_directory(. SOURCES)
add_library(dynlib SHARED ${SOURCES})
target_compile_definitions(dynlib PRIVATE EXPORT_DLL)
target_include_directories(dynlib PUBLIC /usr/local/cuda/include)
target_compile_options(dynlib PRIVATE -fvisibility=hidden -fvisibility-inlines-hidden)
target_link_directories(dynlib PUBLIC /usr/local/cuda/lib64)
target_link_libraries(dynlib PRIVATE cudart)
```
which specifies CUDA architectures with `CMAKE_CUDA_ARCHITECTURES`. CMake would automatically search for cuda headers and libraries if they are installed on system paths. An old way to trigger CUDA compiling behavior is to use `find_package(CUDA)`.


