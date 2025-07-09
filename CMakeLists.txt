cmake_minimum_required(VERSION 3.16)

project(EasyWhisperUI VERSION 0.1 LANGUAGES CXX)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Widgets)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Widgets)

set(PROJECT_SOURCES
        main.cpp
        mainwindow.cpp
        mainwindow.h
        mainwindow.ui
)
if(WIN32)
    list(APPEND PROJECT_SOURCES icon.rc)
endif()

if(${QT_VERSION_MAJOR} GREATER_EQUAL 6)
    qt_add_executable(EasyWhisperUI
        MANUAL_FINALIZATION
        ${PROJECT_SOURCES}
        resources.qrc
        settings.h settings.cpp
        windowhelper.h windowhelper.cpp
        filequeue.h filequeue.cpp
        transcriptionpipeline.h transcriptionpipeline.cpp
    )
# Define target properties for Android with Qt 6 as:
#    set_property(TARGET EasyWhisperUI APPEND PROPERTY QT_ANDROID_PACKAGE_SOURCE_DIR
#                 ${CMAKE_CURRENT_SOURCE_DIR}/android)
# For more information, see https://doc.qt.io/qt-6/qt-add-executable.html#target-creation
else()
    if(ANDROID)
        add_library(EasyWhisperUI SHARED
            ${PROJECT_SOURCES}
        )
# Define properties for Android with Qt 5 after find_package() calls as:
#    set(ANDROID_PACKAGE_SOURCE_DIR "${CMAKE_CURRENT_SOURCE_DIR}/android")
    else()
        add_executable(EasyWhisperUI
            ${PROJECT_SOURCES}
            resources.qrc
            settings.h settings.cpp
            windowhelper.h windowhelper.cpp
            filequeue.h filequeue.cpp
            transcriptionpipeline.h transcriptionpipeline.cpp
        )
    endif()
endif()

target_link_libraries(EasyWhisperUI PRIVATE Qt${QT_VERSION_MAJOR}::Widgets)

# Windows deployment only
if(WIN32)
    find_program(WINDEPLOYQT_EXECUTABLE windeployqt REQUIRED)
    add_custom_command(TARGET EasyWhisperUI POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E make_directory "${CMAKE_SOURCE_DIR}/build/Final"
        COMMAND ${CMAKE_COMMAND} -E copy_if_different "${CMAKE_SOURCE_DIR}/build.bat" "${CMAKE_SOURCE_DIR}/build/Final/build.bat"
        COMMAND ${CMAKE_COMMAND} -E copy_if_different "$<TARGET_FILE:EasyWhisperUI>" "${CMAKE_SOURCE_DIR}/build/Final/EasyWhisperUI.exe"
        COMMAND ${WINDEPLOYQT_EXECUTABLE} --no-opengl-sw --no-opengl --no-quick --no-translations --no-system-d3d-compiler --no-svg --no-network "${CMAKE_SOURCE_DIR}/build/Final/EasyWhisperUI.exe"
        COMMAND ${CMAKE_COMMAND} -E remove_directory "${CMAKE_SOURCE_DIR}/build/Final/imageformats"
        COMMAND ${CMAKE_COMMAND} -E rm -f "${CMAKE_SOURCE_DIR}/build/Final/dxcompiler.dll"
        COMMENT "Windows post-build deployment"
    )
endif()

# Qt for iOS sets MACOSX_BUNDLE_GUI_IDENTIFIER automatically since Qt 6.1.
# If you are developing for iOS or macOS you should consider setting an
# explicit, fixed bundle identifier manually though.
if(${QT_VERSION} VERSION_LESS 6.1.0)
  set(BUNDLE_ID_OPTION MACOSX_BUNDLE_GUI_IDENTIFIER com.example.EasyWhisperUI)
endif()
# Platform-specific bundle/executable properties
if(APPLE)
    set_target_properties(EasyWhisperUI PROPERTIES
        ${BUNDLE_ID_OPTION}
        MACOSX_BUNDLE_BUNDLE_VERSION ${PROJECT_VERSION}
        MACOSX_BUNDLE_SHORT_VERSION_STRING ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}
        MACOSX_BUNDLE TRUE
    )
elseif(WIN32)
    set_target_properties(EasyWhisperUI PROPERTIES
        WIN32_EXECUTABLE TRUE
    )
endif()

include(GNUInstallDirs)
install(TARGETS EasyWhisperUI
    BUNDLE DESTINATION .
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

if(QT_VERSION_MAJOR EQUAL 6)
    qt_finalize_executable(EasyWhisperUI)
endif()
