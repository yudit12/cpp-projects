# CMake minimum version
cmake_minimum_required(VERSION 3.10)

# Project name
project(data_structures_algorithms_cpp)

# Set the C++ standard to C++17
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# Ensure debugging symbols are included
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g")

# Add AddressSanitizer flags for Debug builds
if(CMAKE_BUILD_TYPE MATCHES Debug)
    message(STATUS "Building with AddressSanitizer")
    set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -fsanitize=address -fno-omit-frame-pointer")
    set(CMAKE_LINKER_FLAGS "${CMAKE_LINKER_FLAGS} -fsanitize=address")
endif()

# Add a static library for LinkedList
add_library(LinkedList STATIC LinkedLists/LinkedList.cpp)

# Ensure the main.cpp exists
if(EXISTS "${CMAKE_SOURCE_DIR}/LinkedLists/main.cpp")
    # Add the executable target for the main application
    add_executable(data_structures_algorithms_cpp LinkedLists/main.cpp)
    
    # Link the static library to the main executable
    target_link_libraries(data_structures_algorithms_cpp LinkedList)
else()
    message(FATAL_ERROR "LinkedLists/main.cpp not found!")
endif()

# Add executables for all .cpp files in LinkedLists/QUESTION
file(GLOB QUESTION_SOURCES "${CMAKE_SOURCE_DIR}/LinkedLists/QUESTION/*.cpp")
foreach(source_file ${QUESTION_SOURCES})
    # Extract the base name of the file without extension
    get_filename_component(executable_name ${source_file} NAME_WE)

    # Add executable for each .cpp file
    add_executable(${executable_name} ${source_file})

    # # Optionally, you can link the LinkedList library to each of these executables
    # #This command tells CMake to link a specific library to an executable. This is necessary when the executable depends on code that is implemented in a library.
    # target_link_libraries(${executable_name} LinkedList)

    # Set the output directory for the executables
    set_target_properties(${executable_name} PROPERTIES RUNTIME_OUTPUT_DIRECTORY "${CMAKE_BINARY_DIR}/LinkedLists/QUESTION")
endforeach()


# Install GTest if it's not found
find_package(GTest REQUIRED)
include_directories(${GTEST_INCLUDE_DIRS})

# Add the test executable
if(EXISTS "${CMAKE_SOURCE_DIR}/tests/LinkedListTest.cpp")
    # Add the test executable
    add_executable(LinkedListTest tests/LinkedListTest.cpp)

    # Include the directory containing LinkedList.h for the test executable
    target_include_directories(LinkedListTest PRIVATE LinkedLists)

    # Link the static library to the test executable
    target_link_libraries(LinkedListTest LinkedList)

    # Link GTest to the test executable
    target_link_libraries(LinkedListTest GTest::GTest GTest::Main)

    # # Apply sanitizer flags to LinkedListTest
    # target_compile_options(LinkedListTest PRIVATE -fsanitize=address -fno-omit-frame-pointer)
    # target_link_options(LinkedListTest PRIVATE -fsanitize=address)

else()
    message(FATAL_ERROR "tests/LinkedListTest.cpp not found!")
endif()


# Enable testing
enable_testing()

# Add the test to CTest
add_test(NAME LinkedListTest COMMAND LinkedListTest)

# # Output build type and sanitization status
# message(STATUS "Build Type: ${CMAKE_BUILD_TYPE}")
# message(STATUS "AddressSanitizer: ENABLED")