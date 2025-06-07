cmake_minimum_required(VERSION 3.16)
project(Binairo LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

add_executable(Binairo src/main.cpp)

if(EMSCRIPTEN)
    target_compile_options(Binairo PRIVATE -sUSE_SDL=2)
    target_link_options(Binairo PRIVATE -sUSE_SDL=2 -sASYNCIFY)
    set_target_properties(Binairo PROPERTIES SUFFIX ".html")
elseif(APPLE)
    # macOS
    message(STATUS "Target: macOS")

    # 嘗試使用 Homebrew 的 SDL2
    set(CMAKE_PREFIX_PATH "/opt/homebrew/opt/sdl2;${CMAKE_PREFIX_PATH}")

    # 首先嘗試使用 CONFIG 模式的 find_package
    find_package(SDL2 CONFIG QUIET)

    if(SDL2_FOUND)
        message(STATUS "Found SDL2 via CONFIG mode")
        target_link_libraries(Binairo PRIVATE SDL2::SDL2)
    else()
        # 如果 CONFIG 模式失敗，使用 pkg-config
        find_package(PkgConfig REQUIRED)
        pkg_check_modules(SDL2 REQUIRED sdl2)

        # 手動設置 SDL2 庫路徑
        find_library(SDL2_LIBRARY
                NAMES SDL2
                PATHS /opt/homebrew/lib
                /usr/local/lib
                ${SDL2_LIBRARY_DIRS}
                NO_DEFAULT_PATH
        )

        if(NOT SDL2_LIBRARY)
            message(FATAL_ERROR "SDL2 library not found")
        endif()

        message(STATUS "Found SDL2 library: ${SDL2_LIBRARY}")

        target_include_directories(Binairo PRIVATE ${SDL2_INCLUDE_DIRS})
        target_link_libraries(Binairo PRIVATE ${SDL2_LIBRARY})
        target_compile_options(Binairo PRIVATE ${SDL2_CFLAGS_OTHER})
    endif()

    # OpenGL
    find_package(OpenGL REQUIRED)
    target_link_libraries(Binairo PRIVATE OpenGL::GL)
elseif(WIN32)
    message(STATUS "Target: Windows")
    find_package(SDL2 CONFIG REQUIRED)
    find_package(OpenGL REQUIRED)
    target_link_libraries(Binairo PRIVATE SDL2::SDL2main SDL2::SDL2 opengl32)
elseif(UNIX)
    message(STATUS "Target: Linux")
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(SDL2 REQUIRED sdl2)
    include_directories(${SDL2_INCLUDE_DIRS})
    link_directories(${SDL2_LIBRARY_DIRS})
    add_definitions(${SDL2_CFLAGS_OTHER})
    find_package(OpenGL REQUIRED)
    target_link_libraries(Binairo ${SDL2_LIBRARIES} OpenGL::GL)
else()
    message(FATAL_ERROR "Unknown platform! This CMakeLists only supports macOS, Linux, Windows, Web.")
endif()