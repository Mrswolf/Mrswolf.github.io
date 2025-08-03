---
title: Build Modern C++ Projects in VSCode
date: 2024-01-11 20:43:30
categories: [c++]
permalink: vscode-cpp
tags: [c++]
updated: 2024-01-13 00:00:00
---

<!-- toc -->

I use Visual Studio when I work at the company. Visual Studio does provide a better coding experience on the Windows platform. But honestly, the majority of the coding at the company is just bug fixing, which is less enjoyable. I've made a plan about improving my c++ coding skills since 2024 in my spare time with [my Manjaro system]({% post_url 2019-05-24-manjaro %}). But I don't want to spend too much time diving into details about compiling, linking, etc., at least for now. I want an effortless dev environment to build c++ projects just like Visual Studio does. After some searching, I gotta say, building c++ projects on Linux isn't as hard as it seems at first glance.<!--more-->

## Compilers and Debug Tools
In arch-like systems, installing these tools are easy:
```
sudo pacman -S gcc gdb clang cmake
```

## VScode CPP Extensions
Just as [what I've done for my scientific works]({% post_url 2023-06-11-vscode-python %}), VS code is still my first choice IDE, only with a few more extensions needed for a C++ building environment.

I installed `C/C++ Extension Pack`, containing `C/C++`, `C/C++ Themes`, `CMake`, and `CMake Tools` extensions.

## VSCode CMake Commands
### Create a C++ Project
Open an empty folder and press `Ctrl+Shift+P`, type `CMake: Quick Start` to create a new C++ project. It will prompt you to name your project, choose between a C or C++ project, specify whether it is a library or executable, and finally generate a `CMakeLists.txt`, `main.cpp`, and a `build` folder under the empty parent folder. And here is a good starting point to tweak your C++ project settings.

### Choose a Compiler
Press `Ctrl+Shift+P` and type `CMake: Select a Kit`. It will prompt you to choose compilers installed on your system. In my case, I have `GCC` and `Clang` compilers installed, so I choose `GCC` as my default compiler which i think it's enough for newbies. Don't forget to regenerate your build system with the command `Cmake: Configure`. Interestingly, the output terminal in VS code shows that `CMake` uses `Ninja` instead of `Make` as its default build tool. I have never used `Ninja` before so I thought it's a good opportunity to try this fast, lightweight build tool.

### Compile the Project
Press `Ctrl+Shift+P` and type `CMake: Build` or `F7` to actually build the whole project. If any compiling error happens, you can find it in the bottom Problems panel. The compiled library or executable object would be found under the build folder.

### Switch Release/Debug
VS Code compiles the C++ project in Debug mode by default. Sometime we may switch to Release mode for better optimization. Press `Ctrl+Shift+P`, type `CMake: Select Variant`, and select Release or Debug as it prompts. It would automatically regenerate the build system.

## VSCode CMakePresets
`CMakePresets.json` is another way to tell the build system what settings to use when building the project. It's possible to configure multiple settings for different platforms with this file. Modern IDEs are capable of reading settings from this file, like visual code here.

The file itself is still hard to code correctly by-hands, so we often copy it from github or somewhere else and change it gradually to match our need.

 A typical `CMakePresets.json` file looks like this:

```json
{
    "version": 3,
    "configurePresets": [
        {
            "name": "Debug",
            "description": "Debug settings for GCC",
            "hidden": false,
            "generator": "Ninja",
            "binaryDir": "${workspaceFolder}/build",
            "cacheVariables": {
                "CMAKE_BUILD_TYPE": "Debug",
                "CMAKE_C_COMPILER": "gcc",
                "CMAKE_CXX_COMPILER": "g++",
                "CMAKE_EXPORT_COMPILE_COMMANDS": "YES"
            }
        }
    ]
}

```

