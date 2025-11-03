
#[[

  CMakeLists.txt

  This file is part of fizmo.

  ---

  Copyright (c) 2010-2025 Christoph Ender.
  All rights reserved.

  Redistribution and use in source and binary forms, with or without
  modification, are permitted provided that the following conditions
  are met:
  1. Redistributions of source code must retain the above copyright
  notice, this list of conditions and the following disclaimer.
  2. Redistributions in binary form must reproduce the above copyright
  notice, this list of conditions and the following disclaimer in the
  documentation and/or other materials provided with the distribution.
  3. Neither the name of the copyright holder nor the names of its
  contributors may be used to endorse or promote products derived
  from this software without specific prior written permission.

  THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS
  IS" AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO,
  THE IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR
  PURPOSE ARE DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR
  CONTRIBUTORS BE LIABLE FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL,
  EXEMPLARY, OR CONSEQUENTIAL DAMAGES (INCLUDING, BUT NOT LIMITED TO,
  PROCUREMENT OF SUBSTITUTE GOODS OR SERVICES; LOSS OF USE, DATA, OR
  PROFITS; OR BUSINESS INTERRUPTION) HOWEVER CAUSED AND ON ANY THEORY OF
  LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY, OR TORT (INCLUDING
  NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE OF THIS
  SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

  ---

  This is libfizmo's main buildfile. There are two more in "src/locales"
  and "src/hyph_patterns", both of which are invoked by this one. They
  are needed since they'll have to pre-process the hyphentation patterns
  and locales into ".c" source files, so they can be "hard-compiled" into
  libfizmo, which solves a lot of problem with handling of external files
  and makes debugging in IDEs much simpler.o

  Compile-time options are described individually below.

]]


cmake_minimum_required(VERSION 3.10)

# Enable external commands so it's possible to build the tools which will
# in turn create the ".h" and ".c" files required for the main build first.
include(ExternalProject)

#set(CMAKE_BUILD_TYPE Debug)


project(libfizmo
  LANGUAGES C
  HOMEPAGE_URL https://fizmo.spellbreaker.org
  DESCRIPTION "fizmo interpreter core library"
  VERSION 0.9.0)


# Don't prepend libname with "lib". This is helpful in order to be able
# to distinguish project "fizmo" - the distribution build - from "libfizmo" -
# the core library and have the CMake project names set accordingly.
set(CMAKE_STATIC_LIBRARY_PREFIX "")


# Invoke build and run tool for processing of hyphenation patterns into
# "hyph_patterns.c", so that this file can be used in this build.
ExternalProject_Add(hyph_pattern_preparation
  SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src/hyph_patterns
  BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR}/hyph_patterns_build
  BUILD_COMMAND make
  INSTALL_COMMAND cmake -E echo "Skipping install step."
  )


# Invoke build and run tool for processing of locales into "locale_data.c"
# so that this file can be used in this build.
ExternalProject_Add(locale_data_preparation
  SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src/locales
  BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR}/loacles_build
  BUILD_COMMAND make
  INSTALL_COMMAND cmake -E echo "Skipping install step."
  )


# Provide C math lib during compile.
set(LIBFIZMO_LIBRARIES libfizmo m CACHE INTERNAL "")


# Enabling tracing will cause fizmo to write a really, really large amount
# of debug data to disk while running. Furthermore, after every single
# write, fizmo will invoke an fsync(), meaning that performance will
# quite suffer. This is only useful for debugging and should be disabled
# in all other cases.
option(ENABLE_TRACING "Enable tracing" OFF)
if (ENABLE_TRACING)
  add_definitions(-DENABLE_TRACING)
endif()


# Whether to enable support for the Treaty of Babel, which is described
# at: https://babel.ifarchive.org.
# Since implementing babel requires support for XML, libfizmo build with
# babel support will need the "libxml-2.0" library. If libfizmo is
# supposed to be built without any external dependencies, babel support
# would have to be disabled.
option(ENABLE_BABEL "Enable babel" ON)
if (ENABLE_BABEL)
  add_definitions(-DENABLE_BABEL)
  find_package(LibXml2 REQUIRED)
  set(pc_req_public "libxml-2.0" CACHE INTERNAL "")
  set(LIBFIZMO_REQUIRES "libxml-2.0" CACHE INTERNAL "")
else()
  set(pc_req_public)
endif()


# Enabling STRICT_Z will make fizmo pass strict Z tests, but will make
# some non-perfectly-confirming .z code fail during execution.
option(ENABLE_STRICT_Z "Enable strict Z-machine behavior" OFF)
if (ENABLE_STRICT_Z)
  add_definitions(-DENABLE_STRICT_Z)
endif()


# Enabling segfaults on errors is a "hard fail" feature used only for
# debugging stubborn bugs and shouldn't be enabled.
option(ENABLE_SEGFAULT_ON_ERROR "Enable segfault on error" OFF)
if (ENABLE_SEGFAULT_ON_ERROR)
  add_definitions(-DENABLE_SEGFAULT_ON_ERROR)
endif()


# The "filelist" will remember all games played so far along with their
# location - or games explicitely added by searching - so they can be
# selected from a startup menu when invoking fizmo without story name.
option(ENABLE_FILELIST "Enable filelist" ON)
if (ENABLE_FILELIST)
  add_definitions(-DENABLE_FILELIST)
endif()


# This will enable the "upper window" and should normally be enabled.
option(ENABLE_BLOCKBUFFER "Enable blockbuffer" ON)
if (ENABLE_BLOCKBUFFER)
  add_definitions(-DENABLE_BLOCKBUFFER)
endif()


# Enabling the command history means that fizmo remembers commands
# priviously entered by the user, so that the may be re-used by using
# cursor up/down.
option(ENABLE_COMMAND_HISTORY "Enable command history" ON)
if (ENABLE_COMMAND_HISTORY)
  add_definitions(-DENABLE_COMMAND_HISTORY)
endif()


# The output history will store all output from the code so far, making
# window resize and scrollback possible, which should be enbaled.
option(ENABLE_OUTPUT_HISTORY "Enable output history" ON)
if (ENABLE_OUTPUT_HISTORY)
  add_definitions(-DENABLE_OUTPUT_HISTORY)
endif()


# Whether config files should be implemented and used. Should be enabled
# for normal use.
option(ENABLE_CONFIG_FILES "Enable config files" ON)
if (ENABLE_CONFIG_FILES)
  add_definitions(-DENABLE_CONFIG_FILES)
endif()


# This will disable fizmo's internal commands like "/info".
option(ENABLE_PREFIX_COMMANDS "Enable prefix commands" ON)
if (ENABLE_PREFIX_COMMANDS)
  add_definitions(-DENABLE_PREFIX_COMMANDS)
endif()


# Should always be off at the moment, since the debugger isn't
# implemented.
option(ENABLE_DEBUGGER "Enable debugger" OFF)
if (ENABLE_DEBUGGER)
  add_definitions(-DENABLE_DEBUGGER)
endif()


# In case of a distribution build, this should be set to fizmo's main
# version number, so it's possible to distinguish libfizmo's version
# from the main fizmo version.
if (FIZMO_DIST_VERSION)
  add_definitions(-DFIZMO_DIST_VERSION=${FIZMO_DIST_VERSION})
endif()


# Store filenames of all files-to-be-compiled in "C_FILES".
set (C_FILES
  src/interpreter/babel.c
  src/interpreter/blockbuf.c
  src/interpreter/blorb.c
  src/interpreter/cmd_hst.c
  src/interpreter/config.c
  src/interpreter/debugger.c
  src/interpreter/filelist.c
  src/interpreter/fizmo.c
  src/interpreter/history.c
  src/interpreter/hyphenation.c
  src/interpreter/iff.c
  src/interpreter/mathemat.c
  src/interpreter/misc.c
  src/interpreter/mt19937ar.c
  src/interpreter/object.c
  src/interpreter/output.c
  src/interpreter/property.c
  src/interpreter/routine.c
  src/interpreter/savegame.c
  src/interpreter/sound.c
  src/interpreter/stack.c
  src/interpreter/streams.c
  src/interpreter/table.c
  src/interpreter/text.c
  src/interpreter/undo.c
  src/interpreter/variable.c
  src/interpreter/wordwrap.c
  src/interpreter/zpu.c
  src/tools/filesys.c
  src/tools/filesys_c.c
  src/tools/i18n.c
  src/tools/list.c
  src/tools/stringmap.c
  src/tools/tracelog.c
  src/tools/types.c
  src/tools/z_ucs.c
  src/hyph_patterns/hyph_pattern_tools.c
  src/hyph_patterns/hyph_patterns.c
  src/locales/libfizmo_locales.c
  src/locales/locale_data.c
  src/locales/locale_data.h)


# Tell CMake we'd like to create a library named "libfizmo" with the
# given ".c" files.
add_library(${PROJECT_NAME} ${C_FILES})


# Check whether this is the "top-level" source/project. In case we're
# building libfizmo standalone, we'll install the development files.
if(CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
  # Install the static library. It appears "install" cannot handle a
  # hierarchy of files, hence headers are installed in a separate step.
  install(TARGETS ${PROJECT_NAME}
    ARCHIVE DESTINATION lib COMPONENT Libraries)

  install(DIRECTORY "${PROJECT_SOURCE_DIR}/src/"  # source directory
    DESTINATION "include"                         # target directory
    FILES_MATCHING                                # install only matching files
    PATTERN "*.h")                                # select header files

  set(pc_libs_private)
  set(pc_req_private)
  # "pc_req_public" is set in the "ENABLE_BABEL" section above.
  configure_file(src/libfizmo.pc.in ${PROJECT_NAME}.pc @ONLY)
  install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.pc
    DESTINATION "lib/pkgconfig")
endif()

