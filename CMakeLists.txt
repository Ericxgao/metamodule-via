cmake_minimum_required(VERSION 3.24)

if(NOT "${METAMODULE_SDK_DIR}" STREQUAL "")
	message("METAMODULE_SDK_DIR set by CMake variable ${METAMODULE_SDK_DIR}")
elseif (DEFINED ENV{METAMODULE_SDK_DIR})
    set(METAMODULE_SDK_DIR "$ENV{METAMODULE_SDK_DIR}")
	message("METAMODULE_SDK_DIR set by environment variable ${METAMODULE_SDK_DIR}")
else()
    set(METAMODULE_SDK_DIR "${CMAKE_CURRENT_LIST_DIR}/../../metamodule/metamodule-plugin-sdk")
	message("METAMODULE_SDK_DIR set to default: ${METAMODULE_SDK_DIR}")
endif()
include(${METAMODULE_SDK_DIR}/plugin.cmake)

project(Starling_Via VERSION 1.0.0 DESCRIPTION "Via for Rack" LANGUAGES C CXX)

# Set C++17 standard
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

# Add option for virtual build
option(BUILD_VIRTUAL "Build for virtual environment" ON)

# Define source directories
set(SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/Via-for-Rack)

# Main library target
add_library(Starling_Via STATIC)

# Compiler definitions
target_compile_definitions(Starling_Via PRIVATE 
    __V1x 
    __PLUGIN
    $<$<BOOL:${BUILD_VIRTUAL}>:BUILD_VIRTUAL>
    _MB_CAPABLE=1
)

# Compiler options
target_compile_options(Starling_Via PRIVATE
    -fpermissive
    -fdata-sections
    -ffunction-sections
    -fno-exceptions
    -fno-threadsafe-statics
    -fno-use-cxa-atexit
    $<$<CXX_COMPILER_ID:GNU>:-fvisibility=hidden>
    $<$<CXX_COMPILER_ID:GNU>:-fvisibility-inlines-hidden>
)

# Linker options
target_link_options(Starling_Via PRIVATE
    -Wl,--gc-sections
)

# Source files
file(GLOB_RECURSE MODULE_SOURCES 
    "${SOURCE_DIR}/Via/modules/*.cpp"
)

file(GLOB_RECURSE UI_SOURCES 
    "${SOURCE_DIR}/Via/ui/*.cpp"
)

file(GLOB_RECURSE IO_SOURCES 
    "${SOURCE_DIR}/Via/io/*.cpp"
)

file(GLOB_RECURSE VIA_SOURCES
    "${SOURCE_DIR}/src/*.cpp"
)

# Add custom libc implementations
set(LIBC_SOURCES
    ${METAMODULE_SDK_DIR}/metamodule-plugin-libc/newlib/libc/ctype/jp2uc.c
)

target_sources(Starling_Via PRIVATE
    ${VIA_SOURCES}
    ${MODULE_SOURCES}
    ${UI_SOURCES}
    ${IO_SOURCES}
    ${LIBC_SOURCES}
)

# Include directories
target_include_directories(Starling_Via PRIVATE 
    ${SOURCE_DIR}/src
    ${SOURCE_DIR}/Via/modules/inc
    ${SOURCE_DIR}/Via/ui/inc
    ${SOURCE_DIR}/Via/io/inc
    ${SOURCE_DIR}/src/dep/starling-rack-ui
    ${SOURCE_DIR}/src/dep/starling-dsp
    ${SOURCE_DIR}/Via/modules
    ${METAMODULE_SDK_DIR}/metamodule-plugin-libc/newlib/libc/ctype/jp2uc.c
)

# Create the plugin
create_plugin(
    SOURCE_LIB      Starling_Via
    PLUGIN_NAME     Starling_Via
    PLUGIN_JSON     ${SOURCE_DIR}/plugin.json
    SOURCE_ASSETS   ${CMAKE_CURRENT_LIST_DIR}/assets
    DESTINATION     ${CMAKE_CURRENT_LIST_DIR}/metamodule-plugins
