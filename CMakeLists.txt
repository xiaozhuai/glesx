cmake_minimum_required(VERSION 3.15)
project(glesx)

set(CMAKE_CXX_STANDARD 11)

add_library(EGL INTERFACE)
add_library(GLESv2 INTERFACE)

set(WITH_ANGLE OFF)

if (CMAKE_SYSTEM_NAME STREQUAL Emscripten)
    target_link_options(GLESv2 INTERFACE -sUSE_WEBGL2=1 -sFULL_ES3=1)
elseif (ANDROID)
    # EGL
    find_path(EGL_INCLUDE_DIR EGL/egl.h PATH_SUFFIXES include REQUIRED)
    find_library(EGL_LIB EGL)
    target_include_directories(EGL INTERFACE ${EGL_INCLUDE_DIR})
    target_link_libraries(EGL INTERFACE ${EGL_LIB})
    # GLESv2
    find_path(GLESv3_INCLUDE_DIR GLES3/gl3.h PATH_SUFFIXES include REQUIRED)
    find_library(GLESv3_LIB GLESv3)
    target_include_directories(GLESv2 INTERFACE ${GLESv3_INCLUDE_DIR})
    target_link_libraries(GLESv2 INTERFACE ${GLESv3_LIB})
elseif (UNIX AND NOT APPLE)
    # EGL
    find_path(EGL_INCLUDE_DIR EGL/egl.h PATH_SUFFIXES include REQUIRED)
    find_library(EGL_LIB EGL)
    target_include_directories(EGL INTERFACE ${EGL_INCLUDE_DIR})
    target_link_libraries(EGL INTERFACE ${EGL_LIB})
    # GLESv2
    find_path(GLESv3_INCLUDE_DIR GLES3/gl3.h PATH_SUFFIXES include REQUIRED)
    find_library(GLESv2_LIB GLESv2)
    target_include_directories(GLESv2 INTERFACE ${GLESv3_INCLUDE_DIR})
    target_link_libraries(GLESv2 INTERFACE ${GLESv2_LIB})
elseif (WIN32 OR APPLE)
    set(WITH_ANGLE ON)
    find_package(unofficial-angle CONFIG REQUIRED)
    # EGL
    find_path(EGL_INCLUDE_DIR EGL/egl.h PATH_SUFFIXES include REQUIRED)
    target_include_directories(EGL INTERFACE ${EGL_INCLUDE_DIR})
    target_link_libraries(EGL INTERFACE unofficial::angle::libEGL)
    # GLESv2
    find_path(GLESv3_INCLUDE_DIR GLES3/gl3.h PATH_SUFFIXES include REQUIRED)
    target_include_directories(GLESv2 INTERFACE ${GLESv3_INCLUDE_DIR})
    target_link_libraries(GLESv2 INTERFACE unofficial::angle::libGLESv2)
else ()
    message(FATAL_ERROR "Unsupported platform")
endif ()

add_library(glesx::EGL ALIAS EGL)
add_library(glesx::GLESv2 ALIAS GLESv2)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)
install(TARGETS EGL GLESv2 EXPORT glesx)
install(
        EXPORT glesx
        NAMESPACE glesx::
        DESTINATION "${CMAKE_INSTALL_DATAROOTDIR}/glesx"
        FILE glesx-targets.cmake
)
configure_package_config_file(
        "${CMAKE_CURRENT_LIST_DIR}/glesx-config.cmake.in"
        "${CMAKE_CURRENT_BINARY_DIR}/glesx-config.cmake"
        INSTALL_DESTINATION "${CMAKE_INSTALL_DATAROOTDIR}/glesx"
)
install(
        FILES
        "${CMAKE_CURRENT_BINARY_DIR}/glesx-config.cmake"
        DESTINATION "${CMAKE_INSTALL_DATAROOTDIR}/glesx"
)
