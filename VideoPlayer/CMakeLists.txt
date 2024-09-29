cmake_minimum_required(VERSION 3.12)

project(VideoPlayer VERSION 0.1 LANGUAGES CXX)

set(CMAKE_AUTOUIC ON)
set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTORCC ON)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

find_package(QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Core Quick)
find_package(Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Core Quick)

set(PROJECT_SOURCES
        main.cpp
        qml.qrc
)

include_directories(../libs/)
set(QT_AVPLAYER_DIR ../libs/QtAVPlayer/)
include(../libs/QtAVPlayer/QtAVPlayer.cmake)

set(SOURCES ${QtAVPlayer_SOURCES} main.cpp)


if(${QT_VERSION_MAJOR} GREATER_EQUAL 6)
    qt_add_executable(VideoPlayer
        MANUAL_FINALIZATION
        ${PROJECT_SOURCES}
    )
else()
    if(ANDROID)
        add_library(VideoPlayer SHARED
            ${PROJECT_SOURCES}
        )
    else()
        add_executable(VideoPlayer
          ${PROJECT_SOURCES}
        )
    endif()
endif()


if(NOT WIN32)
    add_definitions(-std=c++1z)
    target_compile_options(${PROJECT_NAME} PRIVATE -fPIC)
endif()


set(LIBS ${QtAVPlayer_LIBS})


if(Qt6_FOUND)
    set(LIBS ${LIBS} Qt6::Core Qt6::Gui Qt6::Multimedia Qt6::Quick Qt6::MultimediaQuickPrivate)
else()
    set(LIBS ${LIBS} Qt5::Core Qt5::Gui Qt5::Quick Qt5::Multimedia ${MultimediaQuick_LIBRARY})
    if (Qt5Core_VERSION VERSION_LESS 5.15.0)
        find_library(MultimediaQuick_LIBRARY REQUIRED NAMES Qt5MultimediaQuick)
        set(LIBS ${LIBS} ${MultimediaQuick_LIBRARY})
    else()
        find_package(Qt5 COMPONENTS MultimediaQuick REQUIRED)
        include_directories(${Qt5MultimediaQuick_PRIVATE_INCLUDE_DIRS})
        set(LIBS ${LIBS} Qt5::MultimediaQuick)
    endif()
endif()


target_link_libraries(VideoPlayer ${LIBS})

set_target_properties(VideoPlayer PROPERTIES
    MACOSX_BUNDLE_GUI_IDENTIFIER my.example.com
    MACOSX_BUNDLE_BUNDLE_VERSION ${PROJECT_VERSION}
    MACOSX_BUNDLE_SHORT_VERSION_STRING ${PROJECT_VERSION_MAJOR}.${PROJECT_VERSION_MINOR}
    MACOSX_BUNDLE TRUE
    WIN32_EXECUTABLE TRUE
)

install(TARGETS VideoPlayer
    BUNDLE DESTINATION .
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR})

if(QT_VERSION_MAJOR EQUAL 6)
    qt_import_qml_plugins(VideoPlayer)
    qt_finalize_executable(VideoPlayer)
endif()
