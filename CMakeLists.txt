cmake_minimum_required(VERSION 3.11)

if(POLICY CMP0077)
    cmake_policy(SET CMP0077 NEW)
endif()

project("sodium" LANGUAGES C)

option(SODIUM_DISABLE_TESTS "Disable tests" OFF)
option(SODIUM_MINIMAL "Only compile the minimum set of functions required for the high-level API" OFF)
option(SODIUM_ENABLE_BLOCKING_RANDOM "Enable this switch only if /dev/urandom is totally broken on the target platform" OFF)

file(GLOB_RECURSE SODIUM_SRC_LIST "libsodium/src/*.c" "libsodium/src/*.h")

add_library(${PROJECT_NAME} ${SODIUM_SRC_LIST})

set_target_properties(${PROJECT_NAME}
    PROPERTIES
        C_STANDARD 99
)

target_include_directories(${PROJECT_NAME}
    PUBLIC
        libsodium/src/libsodium/include
    PRIVATE
        libsodium/src/libsodium/include/sodium
)

target_compile_definitions(${PROJECT_NAME}
    PUBLIC
        $<$<NOT:$<BOOL:${BUILD_SHARED_LIBS}>>:SODIUM_STATIC>
        $<$<BOOL:${SODIUM_MINIMAL}>:SODIUM_LIBRARY_MINIMAL>
    PRIVATE
        CONFIGURED
        $<$<BOOL:${BUILD_SHARED_LIBS}>:SODIUM_DLL_EXPORT>
        $<$<BOOL:${SODIUM_ENABLE_BLOCKING_RANDOM}>:USE_BLOCKING_RANDOM>
        $<$<BOOL:${SODIUM_MINIMAL}>:MINIMAL>
        $<$<C_COMPILER_ID:MSVC>:_CRT_SECURE_NO_WARNINGS>
)

# Variables that need to be exported to version.h.in
set(VERSION 1.0.18)
set(SODIUM_LIBRARY_VERSION_MAJOR 10)
set(SODIUM_LIBRARY_VERSION_MINOR 3)
if(SODIUM_MINIMAL)
    set(SODIUM_LIBRARY_MINIMAL_DEF "#define SODIUM_LIBRARY_MINIMAL 1")
endif()

configure_file(
    libsodium/src/libsodium/include/sodium/version.h.in
    ${CMAKE_CURRENT_SOURCE_DIR}/libsodium/src/libsodium/include/sodium/version.h
)

if(NOT SODIUM_DISABLE_TESTS)
    enable_testing()
    add_subdirectory(test)
endif()
