cmake_minimum_required(VERSION 2.8.8)
project(libffi C)

set(SOURCES_LIST
    src/closures.c
    src/java_raw_api
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

file(COPY ${FFI_CONFIG_FILE} DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
file(COPY src/${TARGETDIR}/ffitarget.h DESTINATION ${CMAKE_CURRENT_BINARY_DIR}/include)

include_directories(${CMAKE_CURRENT_BINARY_DIR}/include)
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
    ${CMAKE_CURRENT_BINARY_DIR}/include/ffi.h
    ${CMAKE_CURRENT_BINARY_DIR}/include/ffitarget.h
    DESTINATION include)
