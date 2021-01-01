cmake_minimum_required(VERSION 3.1.0)
project(hss1374 VERSION 1.0.0)
set(CMAKE_PROJECT_HOMEPAGE_URL "https://www.mixxx.org")
set(CMAKE_PROJECT_DESCRIPTION "High-speed MIDI-over-Firewire device access library for Windows and macOS.")

#
# Build
#

add_library(hss1394
  code/src/1394API.cpp
  code/src/ConfigRom.cpp
  code/src/HSS1394.cpp
)
target_include_directories(hss1394 PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}/code/src")

if(APPLE)
  target_sources(hss1394 PRIVATE code/src/osx/1394API_OSX.cpp)

  find_library(IOKit_LIBRARY IOKit REQUIRED)
  target_link_libraries(hss1394 PUBLIC ${IOKit_LIBRARY})

  find_library(CoreFoundation_LIBRARY CoreFoundation REQUIRED)
  target_link_libraries(hss1394 PUBLIC ${CoreFoundation_LIBRARY})

  find_library(Carbon_LIBRARY IOKit REQUIRED)
  target_link_libraries(hss1394 PUBLIC ${Carbon_LIBRARY})

  if(CMAKE_OSX_DEPLOYMENT_TARGET STREQUAL "10.4u")
    target_compile_definitions(hss1394 PRIVATE "OSX10_4")
  endif()
elseif(WIN32)
    target_compile_definitions(hss1394 PRIVATE "_WIN32_")
else()
    message(FATAL_ERROR "Operating system not supported")
endif()

#
# Installation
#

include(GNUInstallDirs)
install(
  TARGETS hss1394
  DESTINATION "${CMAKE_INSTALL_LIBDIR}"
)

install(
  FILES
    "${CMAKE_CURRENT_SOURCE_DIR}/code/src/HSS1394.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/code/src/HSS1394Types.h"
  DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/hss1394"
)
