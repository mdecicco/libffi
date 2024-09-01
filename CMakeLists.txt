cmake_minimum_required(VERSION 3.0.0)
project(libffi C)

set(SOURCES_LIST
    src/closures.c
    src/java_raw_api.c
    src/prep_cif.c
    src/raw_api.c
    src/types.c
)

if(MSVC)
    list(APPEND SOURCES_LIST src/tramp.c)
endif()

if(CMAKE_BUILD_TYPE MATCHES DEBUG)
    list(APPEND SOURCES_LIST src/debug.c)
    add_definitions(-DFFI_DEBUG)
endif()

include(configure_platform.cmake)
include(configure_options.cmake)

set(FFI_INCLUDE_DIR ${CMAKE_CURRENT_BINARY_DIR}/include CACHE STRING "FFI Include Directory")

file(COPY ${FFI_CONFIG_FILE} DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
file(COPY src/${TARGETDIR}/ffitarget.h DESTINATION ${FFI_INCLUDE_DIR})

include_directories(${FFI_INCLUDE_DIR})
include_directories(${CMAKE_CURRENT_BINARY_DIR})
include_directories(include)

add_definitions(-DFFI_BUILDING)

add_library(ffi_static STATIC ${SOURCES_LIST})

if(MSVC)
    add_definitions(-DFFI_BUILDING_DLL)
    set(CMAKE_WINDOWS_EXPORT_ALL_SYMBOLS ON)
elseif(CMAKE_SYSTEM_NAME MATCHES Darwin)
    set(CMAKE_MACOSX_RPATH 1)
endif()
add_library(ffi_shared SHARED ${SOURCES_LIST})

set_target_properties(ffi_static PROPERTIES OUTPUT_NAME ffi)
set_target_properties(ffi_shared PROPERTIES OUTPUT_NAME ffi)

install(FILES
    ${FFI_INCLUDE_DIR}/ffi.h
    ${FFI_INCLUDE_DIR}/ffitarget.h
    DESTINATION include)
