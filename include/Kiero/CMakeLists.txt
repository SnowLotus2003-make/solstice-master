cmake_minimum_required(VERSION 3.7...3.29)

option(KIERO_INCLUDE_D3D9 "Enable support for D3D9" OFF)
option(KIERO_INCLUDE_D3D10 "Enable support for D3D10" OFF)
option(KIERO_INCLUDE_D3D11 "Enable support for D3D11" ON)
option(KIERO_INCLUDE_D3D12 "Enable support for D3D12" ON)
option(KIERO_INCLUDE_OPENGL "Enable support for OpenGL" OFF)
option(KIERO_INCLUDE_VULKAN "Enable support for Vulkan" OFF)
option(KIERO_USE_MINHOOK "Enables \"kiero::bind\" as a wrapper around MinHook" ON)
option(KIERO_MINHOOK_EXTERNAL "Use an externally provided MinHook instead of the bundled" ON)

project(kiero VERSION 1.2.12)

add_library(kiero STATIC
    src/kiero.cpp
)
target_compile_definitions(kiero PUBLIC KIERO_VERSION=${PROJECT_VERSION})

set_target_properties(kiero PROPERTIES
    CXX_STANDARD 20
    CXX_STANDARD_REQUIRED ON
)

if (MSVC)
    if (CMAKE_CXX_COMPILER_ID MATCHES "Clang")
        target_compile_options(kiero PRIVATE
            /clang:-Wall
            /clang:-Wextra
            /clang:-Wcast-align
            /clang:-Wconversion
            /clang:-Wsign-conversion
            /clang:-Wold-style-cast
            /clang:-Wimplicit-fallthrough
        )
    endif()
    target_compile_options(kiero PRIVATE /WX /W4)
else()
    message(WARNING "Non Microsoft compatible compiler detected")
endif()

if (NOT (KIERO_INCLUDE_D3D9 OR KIERO_INCLUDE_D3D10 OR KIERO_INCLUDE_D3D11 OR KIERO_INCLUDE_D3D12 OR KIERO_INCLUDE_OPENGL OR KIERO_INCLUDE_VULKAN))
    message(FATAL_ERROR "At least one graphics API must be specified")
endif()

if (KIERO_INCLUDE_D3D9)
    target_compile_definitions(kiero PRIVATE KIERO_INCLUDE_D3D9)
endif()

if (KIERO_INCLUDE_D3D10)
    target_compile_definitions(kiero PRIVATE KIERO_INCLUDE_D3D10)
endif()

if (KIERO_INCLUDE_D3D11)
    target_compile_definitions(kiero PRIVATE KIERO_INCLUDE_D3D11)
endif()

if (KIERO_INCLUDE_D3D12)
    target_compile_definitions(kiero PRIVATE KIERO_INCLUDE_D3D12)
endif()

if (KIERO_INCLUDE_OPENGL)
    find_package(OpenGL REQUIRED)
    target_link_libraries(kiero PRIVATE OpenGL::GL)
    target_compile_definitions(kiero PRIVATE KIERO_INCLUDE_OPENGL)
endif()

if (KIERO_INCLUDE_VULKAN)
    find_package(Vulkan REQUIRED)
    target_link_libraries(kiero PRIVATE Vulkan::Vulkan)
    target_compile_definitions(kiero PRIVATE KIERO_INCLUDE_VULKAN)
endif()

if (KIERO_USE_MINHOOK)
    if (NOT KIERO_MINHOOK_EXTERNAL)
        include(FetchContent)
        FetchContent_Declare(
            minhook
            GIT_REPOSITORY https://github.com/TsudaKageyu/minhook.git
            GIT_TAG        f5485b8454544c2f034c78f8f127c1d03dea3636
        )
        FetchContent_MakeAvailable(minhook)
    elseif (NOT TARGET minhook)
        message(FATAL_ERROR "minhook target not present")
    endif()
    target_link_libraries(kiero PRIVATE minhook)
    target_compile_definitions(kiero PRIVATE KIERO_USE_MINHOOK)
endif()

target_include_directories(kiero PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include/>
    $<INSTALL_INTERFACE:include>
)
