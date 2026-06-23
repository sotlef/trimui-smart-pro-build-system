cmake_minimum_required(VERSION 3.16)
project(Trimui-App LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(APP_LABEL       "Trimui-App"                       CACHE STRING "App label shown in launcher")
set(APP_DESCRIPTION "Program for the Trimui Smart Pro" CACHE STRING "App description")
string(REPLACE " " "_" APP_LABEL_SLUG "${APP_LABEL}")

find_package(PkgConfig REQUIRED)
pkg_check_modules(SDL2     REQUIRED IMPORTED_TARGET sdl2)
pkg_check_modules(FREETYPE REQUIRED IMPORTED_TARGET freetype2)
pkg_check_modules(GLESV2   REQUIRED IMPORTED_TARGET glesv2)
pkg_check_modules(EGL      REQUIRED IMPORTED_TARGET egl)

find_library(SDL2_TTF_LIBRARY     NAMES SDL2_ttf REQUIRED)
find_path(SDL2_TTF_INCLUDE_DIR    NAMES SDL_ttf.h PATH_SUFFIXES SDL2 REQUIRED)
add_library(SDL2_ttf::SDL2_ttf UNKNOWN IMPORTED)
set_target_properties(SDL2_ttf::SDL2_ttf PROPERTIES
    IMPORTED_LOCATION             "${SDL2_TTF_LIBRARY}"
    INTERFACE_INCLUDE_DIRECTORIES "${SDL2_TTF_INCLUDE_DIR}")

set(BUNDLE_DIR ${CMAKE_BINARY_DIR}/${APP_LABEL_SLUG})

add_executable(${APP_LABEL_SLUG} src/main.cpp)
target_link_libraries(${APP_LABEL_SLUG} PRIVATE
    PkgConfig::SDL2
    SDL2_ttf::SDL2_ttf
    PkgConfig::FREETYPE
    PkgConfig::GLESV2
    PkgConfig::EGL
    m)
set_target_properties(${APP_LABEL_SLUG} PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY "${BUNDLE_DIR}")

configure_file(${CMAKE_SOURCE_DIR}/cmake/config.json.in
               ${BUNDLE_DIR}/config.json @ONLY)
configure_file(${CMAKE_SOURCE_DIR}/cmake/launch.sh.in
               ${BUNDLE_DIR}/launch.sh   @ONLY)
execute_process(COMMAND chmod +x ${BUNDLE_DIR}/launch.sh)

add_custom_command(TARGET ${APP_LABEL_SLUG} POST_BUILD
    COMMAND ${CMAKE_COMMAND} -E copy
            ${CMAKE_SOURCE_DIR}/res/icon.png
            ${BUNDLE_DIR}/icon.png
    COMMAND ${CMAKE_COMMAND} -E make_directory ${BUNDLE_DIR}/res
    COMMAND ${CMAKE_COMMAND} -E copy
            ${CMAKE_SOURCE_DIR}/res/arial.ttf
            ${BUNDLE_DIR}/res/arial.ttf
    VERBATIM)
