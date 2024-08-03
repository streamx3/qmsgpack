cmake_minimum_required(VERSION 3.16)

# Set the project name and description
project(UnitTests)

# Set C++ standard
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Enable testing
enable_testing()

# Check for Qt6 first, then fall back to Qt5 if not found
find_package(Qt6 QUIET COMPONENTS Core Test)
if(Qt6_FOUND)
    set(QT_USE_QTTEST TRUE)
    set(QT_LIBRARIES Qt6::Core Qt6::Test)
    message(STATUS "Qt6 found")
else()
    find_package(Qt5 REQUIRED COMPONENTS Core Test)
    if(Qt5_FOUND)
        set(QT_USE_QTTEST TRUE)
        set(QT_LIBRARIES Qt5::Core Qt5::Test)
        message(STATUS "Qt5 found")
    else()
        message(FATAL_ERROR "Neither Qt5 nor Qt6 were found")
    endif()
endif()

include(AddFileDependencies)

include_directories(../../src ${CMAKE_CURRENT_BINARY_DIR})

set(UNIT_TESTS qttypes_test)

foreach(test ${UNIT_TESTS})
    message(STATUS "Building ${test}")
    add_executable(${test} ${test}.cpp)

    target_link_libraries(${test}
        ${QT_LIBRARIES}
        qmsgpack
    )

    add_test(NAME ${test} COMMAND ${test})
endforeach()
