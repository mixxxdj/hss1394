cmake_minimum_required(VERSION 3.1.0)
project(TestHSS1394)

find_package(HSS1394 CONFIG REQUIRED)
add_executable(TestHSS1394 TestHSS1394.cpp)
target_link_libraries(TestHSS1394 PRIVATE HSS1394::hss1394)
