# Freetype GL - A C OpenGL Freetype engine
#
# Distributed under the OSI-approved BSD 2-Clause License.  See accompanying
# file `LICENSE` for more details.

cmake_minimum_required(VERSION 2.8.12)

project(freetype-gl)

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} ${SCRIPTS_CMAKE})

set(FREETYPE_VERSION 2.6.5)

message(STATUS "Building for ${CMAKE_SYSTEM_NAME} target system")
message(STATUS "Building with ${CMAKE_C_COMPILER_ID} compiler")

SET(USE_OPENGL OFF CACHE BOOL "Set this to OFF to build a stripped-down version of freetype-gl that doesn't use opengl")

if (USE_OPENGL)
set(freetype-gl_WITH_GLEW_DEFAULT ON)

if((${CMAKE_SYSTEM_NAME} MATCHES "Darwin") OR
    (${CMAKE_SYSTEM_NAME} MATCHES "Android"))
    set(freetype-gl_WITH_GLEW_DEFAULT OFF)
endif()

option(freetype-gl_WITH_GLEW
    "Use the GLEW library to fetch OpenGL function pointers"
    ${freetype-gl_WITH_GLEW_DEFAULT})
option(freetype-gl_USE_VAO "Use a VAO to render a vertex_buffer instance (required for forward compatible OpenGL 3.0 contexts)" OFF)
option(freetype-gl_BUILD_DEMOS "Build the freetype-gl example programs" ON)
option(freetype-gl_BUILD_APIDOC "Build the freetype-gl API documentation" ON)
option(freetype-gl_BUILD_HARFBUZZ "Build the freetype-gl harfbuzz support (experimental)" OFF)
option(freetype-gl_BUILD_MAKEFONT "Build the makefont tool" ON)
option(freetype-gl_BUILD_TESTS "Build the tests" ON)
endif()

if(HAVE_MATH_LIBRARY)
    list(APPEND CMAKE_REQUIRED_LIBRARIES m)
    set(MATH_LIBRARY m)
endif()

if (USE_OPENGL)
if(NOT MINGW AND (WIN32 OR WIN64))
    set(GLEW_ROOT_DIR
        ${CMAKE_CURRENT_SOURCE_DIR}/windows/glew)
    set(ENV{FREETYPE_DIR}
        ${CMAKE_CURRENT_SOURCE_DIR}/windows/freetype)
endif()
find_package(OpenGL REQUIRED)
if(freetype-gl_WITH_GLEW)
    find_package(GLEW REQUIRED)
endif()
endif()

find_package(FREETYPE ${FREETYPE_VERSION} REQUIRED)

include_directories(
    ${FREETYPE_INCLUDE_DIRS}
    ${CMAKE_CURRENT_SOURCE_DIR}
if (USE_OPENGL)
    ${OPENGL_INCLUDE_DIRS}
    ${GLEW_INCLUDE_PATH}
endif()
)

if(MSVC)
    # _CRT_NONSTDC_NO_DEPRECATE -> remove warning C4996
    add_definitions(-D_CRT_SECURE_NO_WARNINGS -D_CRT_NONSTDC_NO_DEPRECATE)
endif(MSVC)

if(freetype-gl_USE_VAO)
    add_definitions(-DFREETYPE_GL_USE_VAO)
endif(freetype-gl_USE_VAO)

if (USE_OPENGL)
    set(FREETYPE_GL_HDR
        distance-field.h
        edtaa3func.h
        font-manager.h
        freetype-gl.h
        markup.h
        opengl.h
        platform.h
        text-buffer.h
        texture-atlas.h
        texture-font.h
        utf8-utils.h
        vec234.h
        vector.h
        vertex-attribute.h
        vertex-buffer.h
    )

    set(FREETYPE_GL_SRC
        distance-field.c
        edtaa3func.c
        font-manager.c
        platform.c
        text-buffer.c
        texture-atlas.c
        texture-font.c
        utf8-utils.c
        vector.c
        vertex-attribute.c
        vertex-buffer.c
    )

add_library(freetype-gl STATIC
    ${FREETYPE_GL_SRC}
    ${FREETYPE_GL_HDR}
)

else()
    set(FREETYPE_GL_HDR
        distance-field.h
        edtaa3func.h
        font-manager.h
        markup.h
        platform.h
        text-buffer.h
        texture-atlas.h
        texture-font.h
        utf8-utils.h
        vec234.h
        vector.h
        version.h
    )

    set(FREETYPE_GL_SRC
        distance-field.c
        edtaa3func.c
        platform.c
        texture-atlas.c
        texture-font.c
        utf8-utils.c
        vector.c
        font-manager.c
    )

add_library(freetype-gl SHARED
    ${FREETYPE_GL_SRC}
    ${FREETYPE_GL_HDR}
)

endif()

if(freetype-gl_BUILD_MAKEFONT)
    add_executable(makefont makefont.c)

    target_link_libraries(makefont
        freetype-gl
        ${OPENGL_LIBRARY}
        ${FREETYPE_LIBRARIES}
        ${MATH_LIBRARY}
        ${GLEW_LIBRARY}
    )

    if(MSVC AND NOT (MSVC_VERSION LESS 1900))
        # prevent error LNK2019: unresolved external symbol _sprintf referenced in function __bdf_parse_properties
        # see http://stackoverflow.com/a/32418900/469659
        target_link_libraries(makefont "legacy_stdio_definitions.lib")
    endif()
endif()

if(freetype-gl_BUILD_APIDOC)
    add_subdirectory(doc)
endif()

if(freetype-gl_BUILD_HARFBUZZ)
    add_subdirectory(harfbuzz)
endif()

if(freetype-gl_BUILD_DEMOS)
    add_subdirectory(demos)
endif()

if(freetype-gl_BUILD_TESTS)
    enable_testing()
    add_subdirectory(tests)
endif(freetype-gl_BUILD_TESTS)

if (NOT USE_OPENGL)
message(STATUS "CMAKE_INSTALL_PREFIX : ${CMAKE_INSTALL_PREFIX}")

install(TARGETS freetype-gl
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
        RUNTIME DESTINATION bin
)

install(FILES
    ${FREETYPE_GL_HDR}
    DESTINATION include/freetype-gl
)

endif()