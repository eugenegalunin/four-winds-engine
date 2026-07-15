cmake_minimum_required(VERSION 3.14)

project(SDL_WINDOW_ENGINE VERSION 20220401.1)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED True)

#set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -std=c++11")
set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} -g -ggdb3 -O0 -Wall -Werror -Wno-sign-compare -fsigned-char -DSWE_DEBUG")
set(CMAKE_CXX_FLAGS_RELEASE "${CMAKE_CXX_FLAGS_RELEASE} -O3 -Wall -Wno-sign-compare -fsigned-char")

option(SWE_SDL12 "switch to SDL12" OFF)
option(SWE_WITH_JSON "add json format support" ON)
option(SWE_WITH_LUA "add lua support" ON)
option(SWE_FOR_LUA_PLUGIN "create shared plugin" OFF)
option(SWE_WITH_ICONV "add encode string support" OFF)
option(SWE_DLOPEN "add load shared libs support, with -ldl" ON)
option(SWE_EXAMPLES "build examples" ON)
option(SWE_DISABLE_AUDIO "disable library SDL_mixer" OFF)
option(SWE_DISABLE_IMAGE "disable library SDL_image" OFF)
option(SWE_DISABLE_TTF "disable library SDL_ttf" OFF)
option(SWE_DISABLE_NETWORK "disable library SDL_net" ON)
option(SWE_DISABLE_TERMGUI "disable build termwin_gui" ON)
option(SWE_DEBUG_SCENE "scene debug hotkeys: (Shift+F8) - highlight windows regions, (Shift+F9) - dump scene to console, (Shift+F10) - dump scene to json" OFF)
option(SWE_WITH_STD_MAP "usage std::unordered, (otherwise flat_hash_map version)" OFF)
option(SWE_DEBUG_MESSAGES "DEBUG messages" ON)
option(SWE_SHARED "shared lib" OFF)
option(SWE_LOG_COLORED "color stdout log" ON)
option(SWE_BUILTIN_ROOTOZOOM2 "built in rotozoom2 (SDL2_gfx)" ON)

include(libswe.cmake)
include_directories(.)

if(SWE_SHARED)
    add_library(libswe SHARED)
else()
    add_library(libswe STATIC)
endif()

target_include_directories(libswe PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${SDL_INCLUDE_DIRS}
    ${SDL2_INCLUDE_DIRS}
    ${SDLGFX_INCLUDE_DIRS}
    ${SDLMIXER_INCLUDE_DIRS}
    ${SDLIMAGE_INCLUDE_DIRS}
    ${SDLTTF_INCLUDE_DIRS}
    ${SDLNET_INCLUDE_DIRS}
    ${PNG_INCLUDE_DIRS}
    ${LZ_INCLUDE_DIRS})

target_link_directories(libswe PUBLIC
    ${SDL_LIBRARY_DIRS}
    ${SDL2_LIBRARY_DIRS}
    ${SDLGFX_LIBRARY_DIRS}
    ${SDLMIXER_LIBRARY_DIRS}
    ${SDLIMAGE_LIBRARY_DIRS}
    ${SDLTTF_LIBRARY_DIRS}
    ${SDLNET_LIBRARY_DIRS}
    ${PNG_LIBRARY_DIRS}
    ${LZ_LIBRARY_DIRS})

# Keep the last pkg-config occurrence so dependent libraries precede their base
# libraries on linkers that still resolve symbols from left to right.
list(REVERSE SWE_LINK_LIBRARIES)
list(REMOVE_DUPLICATES SWE_LINK_LIBRARIES)
list(REVERSE SWE_LINK_LIBRARIES)
target_link_libraries(libswe PUBLIC ${SWE_LINK_LIBRARIES})

target_compile_definitions(libswe PUBLIC
    $<$<BOOL:${SWE_SDL12}>:SWE_SDL12>
    $<$<BOOL:${SWE_WITH_JSON}>:SWE_WITH_JSON>
    $<$<BOOL:${SWE_WITH_LUA}>:SWE_WITH_LUA>
    $<$<BOOL:${SWE_WITH_ICONV}>:SWE_WITH_ICONV>
    $<$<BOOL:${SWE_WITH_STD_MAP}>:SWE_WITH_STD_MAP>
    $<$<BOOL:${SWE_DISABLE_AUDIO}>:SWE_DISABLE_AUDIO>
    $<$<BOOL:${SWE_DISABLE_IMAGE}>:SWE_DISABLE_IMAGE>
    $<$<BOOL:${SWE_DISABLE_TTF}>:SWE_DISABLE_TTF>
    $<$<BOOL:${SWE_DISABLE_NETWORK}>:SWE_DISABLE_NETWORK>
    $<$<BOOL:${SWE_DISABLE_TERMGUI}>:SWE_DISABLE_TERMGUI>
    $<$<BOOL:${SWE_BUILTIN_ROOTOZOOM2}>:SWE_BUILTIN_ROOTOZOOM2>
    $<$<BOOL:${SWE_DLOPEN}>:SWE_DLOPEN>)

add_subdirectory(swe)
set_target_properties(libswe PROPERTIES PREFIX "")

if(SWE_EXAMPLES)
    add_subdirectory(examples)
endif()

if(SWE_WITH_STD_MAP)
    add_compile_options("-DSWE_WITH_STD_MAP")
endif()

if(SWE_LOG_COLORED)
    add_compile_options("-DSWE_LOG_COLORED")
endif()

if(SWE_WITH_LUA)
    pkg_search_module(LUA QUIET lua)

    if(LUA_VERSION VERSION_LESS 5.2)
	include_directories(lua/party/lua52/src)
	set(SWE_WITH_LUA_PARTY ON)
    else()
        add_compile_options(${LUA_CFLAGS})
        add_link_options(${LUA_LDFLAGS_OTHER})
        target_include_directories(libswe PUBLIC ${LUA_INCLUDE_DIRS})
        target_link_directories(libswe PUBLIC ${LUA_LIBRARY_DIRS})
        target_link_libraries(libswe PUBLIC ${LUA_LIBRARIES})
    endif()

    add_compile_options(-DSWE_WITH_LUA)
    add_compile_options(-DLUA_STACK_DEBUG)
    add_subdirectory(lua)
else()
    unset(SWE_FOR_LUA_PLUGIN CACHE)
endif()
