# This file is part of Gramium Desktop,
# a set of libraries for developing nice desktop applications.
#
# For license and copyright information please follow this link:
# https://github.com/GramiumApp/cmake_helpers

add_library(external_quickjs INTERFACE IMPORTED GLOBAL)
add_library(desktop-app::external_quickjs ALIAS external_quickjs)

if (DESKTOP_APP_USE_PACKAGED)
    find_package(PkgConfig QUIET)
    if (PkgConfig_FOUND)
        pkg_check_modules(DESKTOP_APP_QUICKJS IMPORTED_TARGET quickjs)
    endif()
    if (DESKTOP_APP_QUICKJS_FOUND)
        target_link_libraries(external_quickjs INTERFACE PkgConfig::DESKTOP_APP_QUICKJS)
        return()
    endif()
endif()

add_library(external_quickjs_bundled STATIC)
init_target(external_quickjs_bundled "(external)")

set(quickjs_loc ${third_party_loc}/quickjs)

target_sources(external_quickjs_bundled
PRIVATE
    ${quickjs_loc}/quickjs.c
    ${quickjs_loc}/quickjs.h
    ${quickjs_loc}/quickjs-libc.c
    ${quickjs_loc}/quickjs-libc.h
    ${quickjs_loc}/quickjs-atom.h
    ${quickjs_loc}/quickjs-opcode.h
    ${quickjs_loc}/quickjs-c-atomics.h
    ${quickjs_loc}/dtoa.c
    ${quickjs_loc}/dtoa.h
    ${quickjs_loc}/libregexp.c
    ${quickjs_loc}/libregexp.h
    ${quickjs_loc}/libregexp-opcode.h
    ${quickjs_loc}/libunicode.c
    ${quickjs_loc}/libunicode.h
    ${quickjs_loc}/libunicode-table.h
    ${quickjs_loc}/cutils.h
    ${quickjs_loc}/list.h
)

target_include_directories(external_quickjs_bundled
PUBLIC
    ${quickjs_loc}
)

target_compile_definitions(external_quickjs_bundled
PRIVATE
    _GNU_SOURCE
    QUICKJS_NG_BUILD
)

if (WIN32)
    target_compile_definitions(external_quickjs_bundled
    PRIVATE
        WIN32_LEAN_AND_MEAN
        _WIN32_WINNT=0x0601
    )
endif()

set_target_properties(external_quickjs_bundled PROPERTIES
    C_STANDARD 11
    C_STANDARD_REQUIRED ON
    C_EXTENSIONS ON
)

if (MSVC)
    target_compile_options(external_quickjs_bundled
    PRIVATE
        /experimental:c11atomics
        /wd4018
        /wd4100
        /wd4200
        /wd4242
        /wd4244
        /wd4245
        /wd4267
        /wd4388
        /wd4389
        /wd4456
        /wd4457
        /wd4710
        /wd4711
        /wd4820
        /wd4996 
        /wd5045
    )
else()
    target_compile_options(external_quickjs_bundled
    PRIVATE
        -Wno-sign-compare
        -Wno-unused-parameter
        -Wno-unused-but-set-variable
        -Wno-implicit-fallthrough
        -Wno-missing-field-initializers
    )
    if (NOT APPLE)
        target_compile_options(external_quickjs_bundled
        PRIVATE
            -funsigned-char
        )
    endif()
endif()

find_package(Threads)
if (Threads_FOUND)
    target_link_libraries(external_quickjs_bundled PRIVATE Threads::Threads)
endif()
target_link_libraries(external_quickjs_bundled PRIVATE ${CMAKE_DL_LIBS})
find_library(QUICKJS_M_LIBRARY m)
if (QUICKJS_M_LIBRARY)
    target_link_libraries(external_quickjs_bundled PRIVATE m)
endif()
target_link_libraries(external_quickjs
INTERFACE
    external_quickjs_bundled
)
