cmake_minimum_required(VERSION 3.15)
project(glesx)

set(CMAKE_CXX_STANDARD 11)

include(GNUInstallDirs)

set(WITH_ANGLE OFF)

if (CMAKE_SYSTEM_NAME STREQUAL Emscripten)
    # GLESv2
    add_library(GLESv2 INTERFACE)
    target_link_options(GLESv2 INTERFACE -sUSE_WEBGL2=1 -sFULL_ES3=1)
elseif (ANDROID)
    # EGL
    add_library(EGL INTERFACE)
    target_link_options(EGL INTERFACE -lEGL)
    # GLESv2
    add_library(GLESv2 INTERFACE)
    target_link_options(GLESv2 INTERFACE -lGLESv3)
elseif (UNIX AND NOT APPLE)
    # EGL
    add_library(EGL INTERFACE)
    target_link_options(EGL INTERFACE -lEGL)
    # GLESv2
    add_library(GLESv2 INTERFACE)
    target_link_options(GLESv2 INTERFACE -lGLESv2)
elseif (IOS)
    # GLESv2
    install(FILES "ios/include/GLES2/gl2.h" DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/GLES2")
    install(FILES "ios/include/GLES3/gl3.h" DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/GLES3")
    add_library(GLESv2 INTERFACE)
    target_include_directories(GLESv2 INTERFACE
        "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>"
    )
    target_link_libraries(GLESv2 INTERFACE "-framework OpenGLES")
elseif (WIN32 OR APPLE)
    set(WITH_ANGLE ON)
    find_package(unofficial-angle CONFIG REQUIRED)
    # EGL
    add_library(EGL INTERFACE)
    find_path(EGL_INCLUDE_DIR EGL/egl.h PATH_SUFFIXES include REQUIRED)
    target_include_directories(EGL INTERFACE 
        "$<BUILD_INTERFACE:${EGL_INCLUDE_DIR}>"
        "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>"
    )
    target_link_libraries(EGL INTERFACE unofficial::angle::libEGL)
    # GLESv2
    add_library(GLESv2 INTERFACE)
    find_path(GLESv3_INCLUDE_DIR GLES3/gl3.h PATH_SUFFIXES include REQUIRED)
    target_include_directories(GLESv2 INTERFACE 
        "$<BUILD_INTERFACE:${GLESv3_INCLUDE_DIR}>"
        "$<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>"
    )
    target_link_libraries(GLESv2 INTERFACE unofficial::angle::libGLESv2)
else ()
    message(FATAL_ERROR "Unsupported platform")
endif ()

if (TARGET EGL)
    add_library(glesx::EGL ALIAS EGL)
endif ()
add_library(glesx::GLESv2 ALIAS GLESv2)

include(CMakePackageConfigHelpers)
if (TARGET EGL)
    install(TARGETS EGL EXPORT glesx)
endif ()
install(TARGETS GLESv2 EXPORT glesx)
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
