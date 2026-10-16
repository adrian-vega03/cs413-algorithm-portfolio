cmake_minimum_required(VERSION 3.16)
project(PioneerDelivery VERSION 1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

add_library(scheduler src/scheduler.cpp)
target_include_directories(scheduler PUBLIC include)
if(MSVC)
    target_compile_options(scheduler PRIVATE /W4)
else()
    target_compile_options(scheduler PRIVATE -Wall -Wextra -Wpedantic)
endif()

add_executable(pioneer src/main.cpp)
target_link_libraries(pioneer PRIVATE scheduler)

enable_testing()
add_executable(scheduler_tests tests/test_scheduler.cpp)
target_link_libraries(scheduler_tests PRIVATE scheduler)
add_test(NAME scheduler_tests COMMAND scheduler_tests)
add_test(NAME sample_file COMMAND pioneer "${CMAKE_CURRENT_SOURCE_DIR}/data/deliveries.txt")
