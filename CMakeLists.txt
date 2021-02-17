cmake_minimum_required(VERSION 3.4.0)
project(HSS1394 VERSION 1.0.0)
set(CMAKE_PROJECT_HOMEPAGE_URL "https://www.mixxx.org")
set(CMAKE_PROJECT_DESCRIPTION "High-speed MIDI-over-Firewire device access library for Windows and macOS.")


option(BUILD_SHARED_LIBS "Build dynamic library" ON)

include(GNUInstallDirs)

#
# Build
#

add_library(hss1394
  src/1394API.cpp
  src/ConfigRom.cpp
  src/HSS1394.cpp
)
target_include_directories(hss1394 PUBLIC
   $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
   $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

if(APPLE)
  target_sources(hss1394 PRIVATE src/1394API_macOS.cpp)

  find_library(IOKit_LIBRARY IOKit REQUIRED)
  target_link_libraries(hss1394 PUBLIC ${IOKit_LIBRARY})

  find_library(CoreFoundation_LIBRARY CoreFoundation REQUIRED)
  target_link_libraries(hss1394 PUBLIC ${CoreFoundation_LIBRARY})

  find_library(Carbon_LIBRARY Carbon REQUIRED)
  target_link_libraries(hss1394 PUBLIC ${Carbon_LIBRARY})
elseif(WIN32)
  target_sources(hss1394 PRIVATE src/1394API_VHPD1394.cpp)
  target_compile_definitions(hss1394 PRIVATE "_WIN32_")

  # Add header-only library dependency ST1394_ASIO
  target_include_directories(hss1394 SYSTEM PRIVATE lib/ST1394_ASIO)

  # Add library dependency VHPD1304
  add_library(vhpd1394 OBJECT
    lib/VHPD1394/CVhpdAsyncSlave.cpp
    lib/VHPD1394/CVhpdBuf.cpp
    lib/VHPD1394/CVhpd.cpp
    lib/VHPD1394/CVhpdDataSlave.cpp
    lib/VHPD1394/CVhpdNotifySlave.cpp
    lib/VHPD1394/CVhpdThread.cpp
    lib/VHPD1394/SetupApiDll.cpp
  )
  target_include_directories(vhpd1394 SYSTEM PUBLIC lib/VHPD1394)
  target_link_libraries(hss1394 PRIVATE vhpd1394)
  
  set_target_properties(hss1394 PROPERTIES
    WINDOWS_EXPORT_ALL_SYMBOLS TRUE
  )
else()
  message(FATAL_ERROR "Operating system not supported")
endif()

#
# Installation
#

include(CMakePackageConfigHelpers)

# Library files
install(TARGETS hss1394
  EXPORT HSS1394Targets
  ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
  LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
  RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
  INCLUDES DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
)

# Header files
install(
  FILES
    "${CMAKE_CURRENT_SOURCE_DIR}/include/HSS1394.h"
    "${CMAKE_CURRENT_SOURCE_DIR}/include/HSS1394Types.h"
  DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/hss1394"
)

# CMake config
set(HSS1394_INSTALL_CMAKEDIR "lib/cmake/HSS1394")
install(
  EXPORT HSS1394Targets
  FILE HSS1394Targets.cmake
  NAMESPACE HSS1394::
  DESTINATION "${HSS1394_INSTALL_CMAKEDIR}"
)
configure_package_config_file(HSS1394Config.cmake.in
  "${CMAKE_CURRENT_BINARY_DIR}/HSS1394Config.cmake"
  INSTALL_DESTINATION "${HSS1394_INSTALL_CMAKEDIR}"
)
write_basic_package_version_file(
  "${CMAKE_CURRENT_BINARY_DIR}/HSS1394ConfigVersion.cmake"
  VERSION "${CMAKE_PROJECT_VERSION}"
  COMPATIBILITY SameMajorVersion
)
install(
  FILES
    "${CMAKE_CURRENT_BINARY_DIR}/HSS1394Config.cmake"
    "${CMAKE_CURRENT_BINARY_DIR}/HSS1394ConfigVersion.cmake"
  DESTINATION "${HSS1394_INSTALL_CMAKEDIR}"
)
