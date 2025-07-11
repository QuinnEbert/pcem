cmake_minimum_required(VERSION 3.7)

project(PCem)

set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${CMAKE_SOURCE_DIR}/cmake/modules)

include(${CMAKE_SOURCE_DIR}/includes/includes.cmake)
include(${CMAKE_SOURCE_DIR}/cmake/arch-detect.cmake)

target_architecture(PCEM_CPU_TYPE)

set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -O2 -fomit-frame-pointer -fno-strict-aliasing -flto")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O2 -fomit-frame-pointer -fno-strict-aliasing -flto")

set(CMAKE_C_FLAGS_RELWITHDEBINFO "${CMAKE_C_FLAGS_RELEASE} -DDEBUG")
set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "${CMAKE_CXX_FLAGS_RELEASE} -DDEBUG")

if(${PCEM_CPU_TYPE} STREQUAL "i386")
        set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -msse2")
        set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -msse2")
endif()

if(NOT ${PCEM_CPU_TYPE} MATCHES "arm.*")
        set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -mstackrealign")
        set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -mstackrealign")
endif()

if (UNIX)
	add_definitions(-D_FILE_OFFSET_BITS=64)
endif ()

set(PCEM_VERSION_STRING "vNext" CACHE STRING "PCem Version String")
set(PCEM_DISPLAY_ENGINE "wxWidgets" CACHE STRING "PCem Display Engine")
set(PCEM_DEFINES ${PCEM_DEFINES} PCEM_VERSION_STRING="${PCEM_VERSION_STRING}")

message("PCem ${PCEM_VERSION_STRING} - Build Type: ${CMAKE_BUILD_TYPE}")
if(DEFINED CMAKE_TOOLCHAIN_FILE)
        message("Cross Toolchain File: ${CMAKE_TOOLCHAIN_FILE}")
endif()
message("***************************************************************")
message("OS Building For: ${CMAKE_SYSTEM_NAME}")
message("System Processor: ${PCEM_CPU_TYPE}")
message("PCem Display Engine: ${PCEM_DISPLAY_ENGINE}")
message("***************************************************************")

option(PLUGIN_ENGINE "Build PCem with plugin support" ON)
message("Plugin Support: ${PLUGIN_ENGINE}")

option(USE_NETWORKING "Build PCem with networking support" ON)
message("Networking Support: ${USE_NETWORKING}")

if(USE_NETWORKING)
        set(CMAKE_C_FLAGS_RELEASE "${CMAKE_C_FLAGS_RELEASE} -std=gnu17")
        message("       Switching C to gnu17 to allow slirp to build")

        option(USE_PCAP_NETWORKING "Build PCem with PCAP support" ON)
        message("       PCAP Networking Support: ${USE_PCAP_NETWORKING}")
endif()

if(${CMAKE_SYSTEM_NAME} STREQUAL "Linux")
        option(USE_ALSA "Build PCem with MIDI output using ALSA" OFF)
        message("ALSA MIDI Support: ${USE_ALSA}")
        option(FORCE_X11 "Force X11 Mode on Wayland Systems" ON)
        message("Force X11 Mode on Wayland Systems: ${FORCE_X11}")
endif()

option(USE_EXPERIMENTAL "Build PCem with experimental code" OFF)
message("Experimental Modules: ${USE_EXPERIMENTAL}")

if(USE_EXPERIMENTAL)
        option(USE_EXPERIMENTAL_PGC "Build Professional Graphics Controller Support" OFF)
        message("       PGC Support: ${USE_EXPERIMENTAL_PGC}")
        option(USE_EXPERIMENTAL_PRINTER "Build Printer Support" OFF)
        message("       Printer Support: ${USE_EXPERIMENTAL_PRINTER}")
endif()

if(CMAKE_BUILD_TYPE STREQUAL "RelWithDebInfo")
	option(PCEM_RELDEB_AS_RELEASE "Build PCem RelWithDebInfo as Release Mode" OFF)
        message("Build PCem RelWithDebInfo as Release Mode: ${PCEM_RELDEB_AS_RELEASE}")
endif()

if(CMAKE_BUILD_TYPE STREQUAL "Debug" OR (CMAKE_BUILD_TYPE STREQUAL "RelWithDebInfo" AND NOT PCEM_RELDEB_AS_RELEASE))
        message("Extra Debug Messages:")
        option(PCEM_SLIRP_DEBUG "Build PCem with SLIRP_DEBUG debug output" OFF)
        message("       SLIRP_DEBUG debug output: ${PCEM_SLIRP_DEBUG}")
        option(PCEM_RECOMPILER_DEBUG "Build PCem with RECOMPILER_DEBUG debug output" OFF)
        message("       RECOMPILER_DEBUG debug output: ${PCEM_RECOMPILER_DEBUG}")
        option(PCEM_NE2000_DEBUG "Build PCem with NE2000_DEBUG debug output" OFF)
        message("       NE2000_DEBUG debug output: ${PCEM_NE2000_DEBUG}")
        option(PCEM_EMU8K_DEBUG_REGISTERS "Build PCem with EMU8K_DEBUG_REGISTERS debug output" OFF)
        message("       EMU8K_DEBUG_REGISTERS debug output: ${PCEM_EMU8K_DEBUG_REGISTERS}")
        option(PCEM_SB_DSP_RECORD_DEBUG "Build PCem with SB_DSP_RECORD_DEBUG debug output" OFF)
        message("       SB_DSP_RECORD_DEBUG debug output: ${PCEM_SB_DSP_RECORD_DEBUG}")
        option(PCEM_MACH64_DEBUG "Build PCem with MACH64_DEBUG debug output" OFF)
        message("       MACH64_DEBUG debug output: ${PCEM_MACH64_DEBUG}")
        option(PCEM_DEBUG_EXTRA "Build PCem with DEBUG_EXTRA debug output" OFF)
        message("       DEBUG_EXTRA debug output: ${PCEM_DEBUG_EXTRA}")
        option(PCEM_PRINTER_DEBUG "Build PCem with PRINTER_DEBUG debug output" OFF)
        message("       PRINTER_DEBUG debug output: ${PCEM_PRINTER_DEBUG}")
endif()

message("***************************************************************")
if(${CMAKE_SYSTEM_NAME} STREQUAL "Windows")
	set(PCEM_SHARE_DIR "bin" CACHE STRING "PCem Share Directory")
	set(PCEM_PLUGIN_DIR "bin/plugins" CACHE STRING "PCem Plugin Directory")
	set(PCEM_BIN_DIR "bin" CACHE STRING "PCem Binary Directory")
	set(PCEM_LIB_DIR "sdk/lib" CACHE STRING "PCem Library Directory")
	set(PCEM_INCLUDE_DIR "sdk/include" CACHE STRING "PCem Include File Directory")
else()
	set(PCEM_SHARE_DIR "share/pcem" CACHE STRING "PCem Share Directory")
	set(PCEM_PLUGIN_DIR "share/pcem/plugins" CACHE STRING "PCem Plugin Directory")
	set(PCEM_BIN_DIR "bin" CACHE STRING "PCem Binary Directory")
	set(PCEM_LIB_DIR "lib" CACHE STRING "PCem Library Directory")
	set(PCEM_INCLUDE_DIR "include" CACHE STRING "PCem Include File Directory")
endif()
message("PCem Program Directories:")
message("       PCem Prefix Directory: ${CMAKE_INSTALL_PREFIX}")
message("       PCem Share Directory: ${PCEM_SHARE_DIR}")
message("       PCem Binary Directory: ${PCEM_BIN_DIR}")
message("       PCem Library Directory: ${PCEM_LIB_DIR}")
message("       PCem Plugin Directory: ${PCEM_PLUGIN_DIR}")
message("       PCem Include File Directory: ${PCEM_INCLUDE_DIR}")
message("***************************************************************")

set(PCEM_DEFINES ${PCEM_DEFINES} PLUGIN_DIR="${CMAKE_INSTALL_PREFIX}/${PCEM_PLUGIN_DIR}")

include(${CMAKE_SOURCE_DIR}/cmake/debugdefines.cmake)

set(OpenGL_GL_PREFERENCE GLVND)
find_package(SDL2 REQUIRED)
include_directories(${SDL2_INCLUDE_DIRS})

include(${CMAKE_SOURCE_DIR}/cmake/apple.cmake)

find_package(OpenAL REQUIRED)
include_directories(${OPENAL_INCLUDE_DIR})

if(${PCEM_DISPLAY_ENGINE} STREQUAL "wxWidgets")
        find_package(wxWidgets REQUIRED COMPONENTS core base xrc adv)
        include(${wxWidgets_USE_FILE})
        set(DISPLAY_ENGINE_LIBRARIES ${wxWidgets_LIBRARIES})
endif()
if(${PCEM_DISPLAY_ENGINE} STREQUAL "Qt")
        find_package(Qt5 REQUIRED COMPONENTS Core Widgets)
        set(DISPLAY_ENGINE_LIBRARIES Qt5::Core Qt5::Widgets)
        set(CMAKE_AUTOMOC ON)
        set(CMAKE_AUTORCC ON)
        set(CMAKE_AUTOUIC ON)
        include_directories(${Qt5_INCLUDE_DIRS})
endif()


if(USE_ALSA)
        find_package(ALSA REQUIRED)
        include_directories(${ALSA_INCLUDE_DIRS})
endif()

if(USE_NETWORKING AND USE_PCAP_NETWORKING)
        find_package(PCAP REQUIRED)
        include_directories(${PCAP_INCLUDE_DIR})
endif()

find_package(OpenGL REQUIRED)

add_subdirectory(src)

if(USE_EXPERIMENTAL)
        add_subdirectory(experimental)
endif()

include(${CMAKE_SOURCE_DIR}/cmake/install.cmake)
