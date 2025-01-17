cmake_minimum_required(VERSION 3.26)
project(obs-gphoto VERSION 0.4.0)
add_library(${CMAKE_PROJECT_NAME} MODULE)
set(obs-gphoto_DESCRIPTION "Allows connect DSLR cameras with obs-studio through gPhoto on Linux.")
set(CMAKE_BUILD_TYPE Release)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/cmake/modules/")

find_package(libobs REQUIRED)
find_package(Gphoto2 REQUIRED)
find_package(ImageMagick COMPONENTS MagickCore REQUIRED)

find_package(udev)
if(NOT UDEV_FOUND OR DISABLE_UDEV)
    message(STATUS "udev disabled for v4l2 plugin")
else()
    set(gphoto-udev_SOURCES src/gphoto-udev.c src/gphoto-udev.h)
    add_definitions(-DHAVE_UDEV)
endif()

math(EXPR BITS "8*${CMAKE_SIZEOF_VOID_P}")
set(PLUGIN_DIRECTORY "${CMAKE_BINARY_DIR}/build/obs-gphoto")
set(PLUGIN_BIN_DIRECTORY "${PLUGIN_DIRECTORY}/bin/${BITS}bit")

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PLUGIN_BIN_DIRECTORY})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${PLUGIN_BIN_DIRECTORY})

target_sources(
        ${CMAKE_PROJECT_NAME}
        PRIVATE
        src/obs-gphoto.c
        src/gphoto-utils.c
        src/gphoto-utils.h
        ${gphoto-udev_SOURCES}
        src/gphoto-preview.c
        src/gphoto-preview.h
        src/timelapse.c
        src/timelapse.h
)

include(ObsPluginHelpers)

add_definitions(-DMAGICKCORE_QUANTUM_DEPTH=16)
add_definitions(-DMAGICKCORE_HDRI_ENABLE=0)

target_include_directories(
        ${CMAKE_PROJECT_NAME}
        PRIVATE ${CMAKE_SOURCE_DIR}/src
)


target_link_libraries(
        ${CMAKE_PROJECT_NAME}
        PRIVATE OBS::libobs
        PRIVATE ImageMagick::MagickCore
        PRIVATE ${Gphoto2_LIBRARIES}
        PRIVATE ${UDEV_LIBRARIES}
)
target_compile_options(${CMAKE_PROJECT_NAME} PRIVATE -Wall)

if(NOT SYSTEM_INSTALL)
    set(CMAKE_INSTALL_PREFIX $ENV{HOME}/.config/)
    set(OBS_PLUGIN_DESTINATION obs-studio/plugins/${CMAKE_PROJECT_NAME}/bin/${_ARCH_SUFFIX}bit)
endif()

setup_plugin_target(${CMAKE_PROJECT_NAME})


