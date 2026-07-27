cmake_minimum_required(VERSION 3.8...3.28)

project(usvr C CXX)

set(CMAKE_SYSTEM_NAME "Xenon")

# Toolchain modules
set(CMAKE_XENON_TOOLCHAIN_DIR "${CMAKE_CURRENT_SOURCE_DIR}/extern/xbox-common/extern/cmake-xenon-toolchain")
include("${CMAKE_XENON_TOOLCHAIN_DIR}/cmake/config.cmake")
include("${CMAKE_XENON_TOOLCHAIN_DIR}/cmake/compiler.cmake")

# Include directories
include_directories(
    "${CMAKE_CURRENT_SOURCE_DIR}/include"
    "${CMAKE_CURRENT_SOURCE_DIR}/extern/xbox-common/xkelib"
)

# Preprocessor definitions
add_definitions(-D_LIB)

# Project sources
set(USVR_SOURCES
    "${CMAKE_CURRENT_SOURCE_DIR}/src/logging.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/main.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/nand.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/nandsvr.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/svr.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/sysex.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/udpannc.cpp"
    "${CMAKE_CURRENT_SOURCE_DIR}/src/util.cpp"
)

# Add Xenon Module (Dynamic Library / System Module) using xex/xex.xml
add_xenon_module(usvr "${CMAKE_CURRENT_SOURCE_DIR}/xex/xex.xml" "${USVR_SOURCES}")

# Additional Link Options for usvr (DLL entry point, base address, module definition)
target_link_options(usvr PRIVATE
    "/ENTRY:_DllMainCRTStartup"
    "/BASE:0x9A000000"
    "/DEF:${CMAKE_CURRENT_SOURCE_DIR}/xex/usvr.def"
)

# Post-build command using xextool
set(XEXTOOL_EXE "${CMAKE_CURRENT_SOURCE_DIR}/extern/xbox-common/bin/xextool.exe")

add_custom_command(
    TARGET usvr POST_BUILD
    COMMAND "${XEXTOOL_EXE}" -m r -e d -r a -o "$<TARGET_FILE_DIR:usvr>/usvr.xex" "$<TARGET_FILE:usvr>"
    COMMENT "Running xextool to generate retail usvr.xex without encryption..."
)
