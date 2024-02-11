cmake_minimum_required(VERSION 3.15)
include(FeatureSummary)

project(chatterino-crash-handler 
        VERSION 0.1.0
        LANGUAGES CXX
)

option(CRASH_HANDLER_BUILD_TESTS "Build tests for the crash handler" OFF)

add_subdirectory("${CMAKE_CURRENT_LIST_DIR}/lib/crashpad" EXCLUDE_FROM_ALL)
add_subdirectory("${CMAKE_CURRENT_LIST_DIR}/lib/magic_enum" EXCLUDE_FROM_ALL)

set(PROJECT_LIB "${PROJECT_NAME}-lib")

set(LIB_SOURCE_FILES
    src/CommandLine.hpp
    src/CommandLine.cpp
    src/Recovery.hpp
    src/Recovery.cpp
)

set(EXE_SOURCE_FILES
    src/main.cpp
)

if(WIN32)
    list(APPEND LIB_SOURCE_FILES
        src/WinSupport.hpp
        src/WinSupport.cpp
    )
endif()

add_library(${PROJECT_LIB} OBJECT ${LIB_SOURCE_FILES})
target_link_libraries(${PROJECT_LIB}
    PUBLIC
        crashpad_handler_lib
        crashpad_tools
        magic_enum::magic_enum
)
target_include_directories(${PROJECT_LIB} PUBLIC "${CMAKE_CURRENT_LIST_DIR}/src")
set_target_properties(${PROJECT_LIB} PROPERTIES CXX_STANDARD 20)

add_executable(${PROJECT_NAME} WIN32 ${EXE_SOURCE_FILES})
target_link_libraries(${PROJECT_NAME} PRIVATE ${PROJECT_LIB})
set_target_properties(${PROJECT_NAME}
    PROPERTIES
        RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin/crashpad"
        OUTPUT_NAME "crashpad-handler"
        CXX_STANDARD 20
)
install(TARGETS ${PROJECT_NAME} RUNTIME DESTINATION chrashpad)

if(WIN32)
    target_compile_definitions(${PROJECT_LIB} PUBLIC NOMINMAX WIN32_LEAN_AND_MEAN)
endif()

# Configure compilers
if(MSVC)
    target_compile_options(${PROJECT_LIB} PUBLIC /EHsc /bigobj /utf-8)
    target_compile_options(${PROJECT_LIB} PUBLIC
        /W4
        # 5038 - warnings about initialization order
        /w15038
    )
else()
    message(WARNING "No warnings configured for this compiler!")
endif()

if(CHATTERINO_ENABLE_LTO)
    message(STATUS "Enabling LTO for ${PROJECT_NAME}")
    set_property(TARGET ${PROJECT_NAME}
                 PROPERTY INTERPROCEDURAL_OPTIMIZATION TRUE)
endif()

# TESTS

if(CRASH_HANDLER_BUILD_TESTS)
    message(STATUS "Tests enabled for ${PROJECT_NAME}")

    include(GoogleTest)

    set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)

    add_subdirectory("${CMAKE_CURRENT_LIST_DIR}/lib/googletest" EXCLUDE_FROM_ALL)

    enable_testing()

    set(PROJECT_TESTS "${PROJECT_NAME}-test")
    set(TEST_FILES 
        tests/CommandLineTest.cpp
        tests/RecoveryTest.cpp
    )
    if(WIN32)
        list(APPEND TEST_FILES
            tests/WinSupportTest.cpp
        )
    endif()

    add_executable(${PROJECT_TESTS} ${TEST_FILES})
    target_link_libraries(${PROJECT_TESTS} PRIVATE ${PROJECT_LIB} GTest::gtest_main gmock)
    set_target_properties(${PROJECT_TESTS}
        PROPERTIES
            RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/bin"
            CXX_STANDARD 20
    )
    gtest_discover_tests(${PROJECT_TESTS})
endif()
