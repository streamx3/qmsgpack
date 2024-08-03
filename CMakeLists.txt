cmake_minimum_required(VERSION 3.16.0)

project(qmsgpack)

set(CMAKE_CXX_STANDARD 11)

set(CMAKE_INSTALL_NAME_DIR ${LIB_INSTALL_DIR})

# build type
if("${CMAKE_BUILD_TYPE}" MATCHES "^Rel.*")
    add_definitions("-DQT_NO_DEBUG_OUTPUT")
endif("${CMAKE_BUILD_TYPE}" MATCHES "^Rel.*")

# 'd' postfix on windows if debug
if(WIN32)
    set(CMAKE_DEBUG_POSTFIX "d")
endif(WIN32)

option(QT4_BUILD "Build with Qt4" OFF)

if(QT4_BUILD)
    find_package(Qt4 REQUIRED)
    include(${QT_USE_FILE})
    set(CMAKE_INCLUDE_CURRENT_DIR ON)
    set(CMAKE_AUTOMOC ON)
    set(PC_Requires "QtCore")
else()
    find_package(Qt6 QUIET COMPONENTS Core)

    if(Qt6Core_FOUND)
        message("Qt6 found")
        include_directories(${Qt6Core_INCLUDE_DIRS})
        add_definitions(${Qt6Core_DEFINITIONS})
        set(CMAKE_INCLUDE_CURRENT_DIR ON)
        set(CMAKE_AUTOMOC ON)
        set(CMAKE_AUTORCC ON)
        set(CMAKE_AUTOUIC ON)
        set(PC_Requires "Qt6Core")
    else()
        find_package(Qt5 REQUIRED COMPONENTS Core)

        if(Qt5Core_FOUND)
            message("Qt5 found")
            include_directories(${Qt5Core_INCLUDE_DIRS})
            add_definitions(${Qt5Core_DEFINITIONS})
            set(CMAKE_INCLUDE_CURRENT_DIR ON)
            set(CMAKE_AUTOMOC ON)
            set(CMAKE_AUTORCC ON)
            set(CMAKE_AUTOUIC ON)
            set(PC_Requires "Qt5Core")
        else()
            message(FATAL_ERROR "Qt5 or Qt6 not found")
        endif()
    endif()
endif()

option(WITH_GUI_TYPES "Build with support for QtGui types")

if(WITH_GUI_TYPES)
    if(QT4_BUILD)
        find_package(Qt4 COMPONENTS QTGUI REQUIRED)
    else()
        if(Qt6Core_FOUND)
            find_package(Qt6 COMPONENTS Gui REQUIRED)
        else()
            find_package(Qt5 COMPONENTS Gui REQUIRED)
        endif()
    endif()
endif()

option(WITH_LOCATION_TYPES "Build with support for QtLocation types")

if(WITH_LOCATION_TYPES)
    if(Qt6Core_FOUND)
        find_package(Qt6 COMPONENTS Location REQUIRED)
    else()
        find_package(Qt5 COMPONENTS Location REQUIRED)
    endif()
endif()

if(Qt5Location_FOUND OR Qt6Location_FOUND)
    if(Qt5Location_FOUND)
        message("Qt5Location found")
        include_directories(${Qt5Location_INCLUDE_DIRS})
        add_definitions(${Qt5Location_DEFINITIONS})
    else()
        message("Qt6Location found")
        include_directories(${Qt6Location_INCLUDE_DIRS})
        add_definitions(${Qt6Location_DEFINITIONS})
    endif()
else()
    message("QtLocation not found")
endif()

if(NOT WIN32)
    set(QT_DONT_USE_QTGUI TRUE)
endif()

# add extra search paths for libraries and includes
set(LIB_SUFFIX "" CACHE STRING "Define suffix of directory name (32/64)")
set(LIB_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}/lib${LIB_SUFFIX}" CACHE STRING "Directory where lib will install")
set(INCLUDE_INSTALL_DIR "${CMAKE_INSTALL_PREFIX}/include" CACHE PATH "The directory the headers are installed in")
set(CMAKECONFIG_INSTALL_DIR "${LIB_INSTALL_DIR}/cmake/${CMAKE_PROJECT_NAME}" CACHE PATH "Directory where to install qmsgpack.cmake")

configure_file(
    "${CMAKE_CURRENT_SOURCE_DIR}/cmake_uninstall.cmake.in"
    "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
    IMMEDIATE @ONLY)

set(QMSGPACK_MAJOR "0")
set(QMSGPACK_MINOR "1")
set(QMSGPACK_VERSION "0")

set(MSGPACK_QT_LIB_VERSION_STRING "${QMSGPACK_MAJOR}.${QMSGPACK_MINOR}.${QMSGPACK_VERSION}")

set(CMAKE_LIBRARY_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/lib")

add_subdirectory(src)

if(BUILD_TESTS)
    enable_testing()
    add_subdirectory(tests)
endif()

install(EXPORT qmsgpack-export DESTINATION ${CMAKECONFIG_INSTALL_DIR} FILE MsgPackQtTargets.cmake)

file(RELATIVE_PATH relInstallDir ${CMAKE_INSTALL_PREFIX}/$CMAKECONFIG_INSTALL_DIR} ${CMAKE_INSTALL_PREFIX})

add_custom_target(uninstall
    "${CMAKE_COMMAND}" -P "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake")
