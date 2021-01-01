cmake_minimum_required(VERSION 3.1.0)
project(hss1394 VERSION 1.0.0)
set(CMAKE_PROJECT_HOMEPAGE_URL "https://www.mixxx.org")
set(CMAKE_PROJECT_DESCRIPTION "High-speed MIDI-over-Firewire device access library for Windows and macOS.")

option(BUILD_SHARED_LIBS "Build dynamic library" ON)

#
# Build
#

add_library(hss1394
  src/1394API.cpp
  src/ConfigRom.cpp
  src/HSS1394.cpp
)
target_include_directories(hss1394 PRIVATE
  "${CMAKE_CURRENT_SOURCE_DIR}/include"
  "${CMAKE_CURRENT_SOURCE_DIR}/src"
)

if(APPLE)
  target_sources(hss1394 PRIVATE src/osx/1394API_OSX.cpp)

  find_library(IOKit_LIBRARY IOKit REQUIRED)
  target_link_libraries(hss1394 PUBLIC ${IOKit_LIBRARY})

  find_library(CoreFoundation_LIBRARY CoreFoundation REQUIRED)
  target_link_libraries(hss1394 PUBLIC ${CoreFoundation_LIBRARY})

  find_library(Carbon_LIBRARY Carbon REQUIRED)
  target_link_libraries(hss1394 PUBLIC ${Carbon_LIBRARY})

  if(CMAKE_OSX_DEPLOYMENT_TARGET STREQUAL "10.4u")
    target_compile_definitions(hss1394 PRIVATE "OSX10_4")
  endif()
elseif(WIN32)
  target_sources(hss1394 PRIVATE src/win32/1394API_VHPD1394.cpp)
  target_compile_definitions(hss1394 PRIVATE "_WIN32_")
  if(BUILD_SHARED_LIBS)
    target_compile_definitions(hss1394 PRIVATE "HSS1394_EXPORT_DLL")
  endif()

  # Add header-only library dependency ST1394_ASIO
  target_include_directories(hss1394 SYSTEM PRIVATE src/win32/Thesycon/ST1394_ASIO)

  # Add library dependency VHPD1304
  add_library(vhpd1394 OBJECT
    src/win32/Thesycon/VHPD1394/CVhpdAsyncSlave.cpp
    src/win32/Thesycon/VHPD1394/CVhpdBuf.cpp
    src/win32/Thesycon/VHPD1394/CVhpd.cpp
    src/win32/Thesycon/VHPD1394/CVhpdDataSlave.cpp
    src/win32/Thesycon/VHPD1394/CVhpdNotifySlave.cpp
    src/win32/Thesycon/VHPD1394/CVhpdThread.cpp
    src/win32/Thesycon/VHPD1394/SetupApiDll.cpp
  )
  target_include_directories(vhpd1394 SYSTEM PUBLIC src/win32/Thesycon/VHPD1394)
  target_link_libraries(hss1394 PRIVATE vhpd1394)
else()
    message(FATAL_ERROR "Operating system not supported")
endif()

#
# Installation
#

include(GNUInstallDirs)

install(TARGETS hss1394
  ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
  LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
  RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
)


install(
  FILES
    "${CMAKE_CURRENT_SOURCE_DIR}/include/HSS1394.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/include/HSS1394Types.h"
  DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/hss1394"
)
