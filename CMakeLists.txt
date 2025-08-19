CMAKE_MINIMUM_REQUIRED(VERSION 3.25)
PROJECT(libtins)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

set(TINS_HAVE_CXX11 ON) # tins flags

# Compile in release mode by default
IF(NOT CMAKE_BUILD_TYPE)
    MESSAGE(STATUS "Setting build type to 'RelWithDebInfo' as none was specified.")
    SET(CMAKE_BUILD_TYPE RelWithDebInfo)
ELSE(NOT CMAKE_BUILD_TYPE)
    MESSAGE(STATUS "Using specified '${CMAKE_BUILD_TYPE}' build type.")
ENDIF(NOT CMAKE_BUILD_TYPE)

SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -Wextra")

# Build output checks
OPTION(LIBTINS_BUILD_SHARED "Build libtins as a shared library." ON)
IF(LIBTINS_BUILD_SHARED)
    MESSAGE(
        STATUS
        "Build will generate a shared library. "
        "Use LIBTINS_BUILD_SHARED=0 to perform a static build"
    )
    SET(LIBTINS_TYPE SHARED)
ELSE(LIBTINS_BUILD_SHARED)
    MESSAGE(STATUS "Build will generate a static library.")
    SET(LIBTINS_TYPE STATIC)
    ADD_DEFINITIONS("-DTINS_STATIC=1")
ENDIF(LIBTINS_BUILD_SHARED)

# The version number.
SET(TINS_VERSION_MAJOR 4)
SET(TINS_VERSION_MINOR 6)
SET(TINS_VERSION_PATCH 0)
SET(LIBTINS_VERSION "${TINS_VERSION_MAJOR}.${TINS_VERSION_MINOR}")

# Required Packages
SET(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_CURRENT_SOURCE_DIR}")

# Allow disabling packet capture mechanism
OPTION(LIBTINS_ENABLE_PCAP "Enable capturing packets via libpcap" ON)

# Look for libpcap
IF(LIBTINS_ENABLE_PCAP)
    FIND_PACKAGE(PCAP REQUIRED)
    SET(TINS_HAVE_PCAP ON)
ENDIF()

# *******************
# Compilation options
# *******************

IF(HAS_GCC_BUILTIN_SWAP)
    SET(TINS_HAVE_GCC_BUILTIN_SWAP ON)
ENDIF()

# IEEE 802.11 and WPA2 decryption support
OPTION(LIBTINS_ENABLE_DOT11 "Compile libtins with IEEE 802.11 support" ON)
OPTION(LIBTINS_ENABLE_WPA2 "Compile libtins with WPA2 decryption features (requires OpenSSL)" ON)
IF(LIBTINS_ENABLE_DOT11)
    SET(TINS_HAVE_DOT11 ON)
    MESSAGE(STATUS "Enabling IEEE 802.11 support.")
    IF(LIBTINS_ENABLE_WPA2)
        FIND_PACKAGE(OpenSSL)
        IF(OPENSSL_FOUND)
            SET(TINS_HAVE_WPA2_DECRYPTION ON)
            MESSAGE(STATUS "Enabling WPA2 decryption support.")
        ELSE()
            MESSAGE(WARNING "Disabling WPA2 decryption support since OpenSSL was not found")
            # Default this to empty strings
            SET(OPENSSL_INCLUDE_DIR "")
            SET(OPENSSL_LIBRARIES "")
        ENDIF()
    ELSE(LIBTINS_ENABLE_WPA2)
        MESSAGE(STATUS "Disabling WPA2 decryption support.")
    ENDIF(LIBTINS_ENABLE_WPA2)
ENDIF(LIBTINS_ENABLE_DOT11)

# Optionally enable TCPIP classes (on by default)
OPTION(LIBTINS_ENABLE_TCPIP "Enable TCPIP classes" ON)
IF(LIBTINS_ENABLE_TCPIP AND TINS_HAVE_CXX11)
    SET(TINS_HAVE_TCPIP ON)
    MESSAGE(STATUS "Enabling TCPIP classes")
ELSE()
    SET(TINS_HAVE_TCPIP OFF)
    MESSAGE(STATUS "Disabling TCPIP classes")
ENDIF()


# 依赖 Boost 的功能, 暂时不用
SET(TINS_HAVE_ACK_TRACKER OFF)
SET(TINS_HAVE_TCP_STREAM_CUSTOM_DATA OFF)

OPTION(LIBTINS_ENABLE_WPA2_CALLBACKS "Enable WPA2 callback interface" ON)
IF(LIBTINS_ENABLE_WPA2_CALLBACKS AND TINS_HAVE_WPA2_DECRYPTION AND TINS_HAVE_CXX11)
    SET(STATUS "Enabling WPA2 callback interface")
    SET(TINS_HAVE_WPA2_CALLBACKS ON)
ENDIF()

# Use pcap_sendpacket to send l2 packets rather than raw sockets
IF(WIN32)
    SET(USE_PCAP_SENDPACKET_DEFAULT ON)
ELSE(WIN32)
    SET(USE_PCAP_SENDPACKET_DEFAULT OFF)
ENDIF(WIN32)

OPTION(LIBTINS_USE_PCAP_SENDPACKET "Use pcap_sendpacket to send l2 packets"
        ${USE_PCAP_SENDPACKET_DEFAULT})
IF(LIBTINS_ENABLE_PCAP AND LIBTINS_USE_PCAP_SENDPACKET)
    SET(TINS_HAVE_PACKET_SENDER_PCAP_SENDPACKET ON)
    MESSAGE(STATUS "Using pcap_sendpacket to send l2 packets.")
ENDIF()


# Configuration file
CONFIGURE_FILE(
  "${PROJECT_SOURCE_DIR}/include/tins/config.h.in"
  "${PROJECT_SOURCE_DIR}/include/tins/config.h"
)

# The library output directory
SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/lib)
SET(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${PROJECT_BINARY_DIR}/bin)


# ******************
# Add subdirectories
# ******************
ADD_SUBDIRECTORY(src)

