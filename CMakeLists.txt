cmake_minimum_required(VERSION 3.31)
project(Binairo)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(SDL2_ROOT /opt/homebrew/opt/sdl2)
set(CMAKE_PREFIX_PATH ${SDL2_ROOT})

find_package(PkgConfig REQUIRED)
pkg_check_modules(SDL2 REQUIRED sdl2)

include_directories(${SDL2_INCLUDE_DIRS})
link_directories(${SDL2_LIBRARY_DIRS})
add_definitions(${SDL2_CFLAGS_OTHER})

find_library(OpenGL_LIBRARY OpenGL)
include_directories(/System/Library/Frameworks/OpenGL.framework/Headers)

add_executable(Binairo main.cpp)

target_link_libraries(Binairo ${SDL2_LIBRARIES} ${OpenGL_LIBRARY})
