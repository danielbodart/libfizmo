
#[[

  CMakeLists.txt

  This file is part of fizmo.

  ---

  Copyright (c) 2010-2023 Christoph Ender.
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



  This is libfizmo's main buildfile. […]

]]


cmake_minimum_required(VERSION 3.10)

include(ExternalProject)

#set(CMAKE_BUILD_TYPE Debug)

project(libfizmo
  LANGUAGES C
  DESCRIPTION "fizmo interpreter core library"
  VERSION 0.9.0)

# Compile hyphenation patterns into "hyph_patterns.c".
ExternalProject_Add(hyph_pattern_preparation
  SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src/hyph_patterns
  BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR}/hyph_patterns_build
  BUILD_COMMAND make
  INSTALL_COMMAND cmake -E echo "Skipping install step."
  )

# Compile locale data into "locale_data.c".
ExternalProject_Add(locale_data_preparation
  SOURCE_DIR ${CMAKE_CURRENT_SOURCE_DIR}/src/locales
  BINARY_DIR ${CMAKE_CURRENT_BINARY_DIR}/loacles_build
  BUILD_COMMAND make
  INSTALL_COMMAND cmake -E echo "Skipping install step."
  )

option(ENABLE_TRACING "Enable tracing" OFF)
if (ENABLE_TRACING)
  add_definitions(-DENABLE_TRACING)
endif()

option(ENABLE_BABEL "Enable babel" ON)
if (ENABLE_BABEL)
  add_definitions(-DENABLE_BABEL)
  find_package(LibXml2 REQUIRED)
  set(pc_req_public "libxml-2.0")
else()
  set(pc_req_public)
endif()

option(ENABLE_STRICT_Z "Enable strict Z-machine behavior" OFF)
if (ENABLE_STRICT_Z)
  add_definitions(-DENABLE_STRICT_Z)
endif()

option(ENABLE_SEGFAULT_ON_ERROR "Enable segfault on error" OFF)
if (ENABLE_SEGFAULT_ON_ERROR)
  add_definitions(-DENABLE_SEGFAULT_ON_ERROR)
endif()

option(ENABLE_FILELIST "Enable filelist" ON)
if (ENABLE_FILELIST)
  add_definitions(-DENABLE_FILELIST)
endif()

option(ENABLE_BLOCKBUFFER "Enable blockbuffer" ON)
if (ENABLE_BLOCKBUFFER)
  add_definitions(-DENABLE_BLOCKBUFFER)
endif()

option(ENABLE_COMMAND_HISTORY "Enable command history" ON)
if (ENABLE_COMMAND_HISTORY)
  add_definitions(-DENABLE_COMMAND_HISTORY)
endif()

option(ENABLE_OUTPUT_HISTORY "Enable output history" ON)
if (ENABLE_OUTPUT_HISTORY)
  add_definitions(-DENABLE_OUTPUT_HISTORY)
endif()

option(ENABLE_CONFIG_FILES "Enable config files" ON)
if (ENABLE_CONFIG_FILES)
  add_definitions(-DENABLE_CONFIG_FILES)
endif()

option(ENABLE_PREFIX_COMMANDS "Enable prefix commands" ON)
if (ENABLE_PREFIX_COMMANDS)
  add_definitions(-DENABLE_PREFIX_COMMANDS)
endif()

option(ENABLE_DEBUGGER "Enable debugger" OFF)
if (ENABLE_DEBUGGER)
  add_definitions(-DENABLE_DEBUGGER)
endif()

if (FIZMO_DIST_VERSION)
  add_definitions(-DFIZMO_DIST_VERSION=${FIZMO_DIST_VERSION})
endif()

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

set (H_FILES
  src/blorb_interface/blorb_interface.h
  src/filesys_interface/filesys_interface.h
  src/hyph_patterns/hyph_pattern_tools.h
  src/hyph_patterns/hyph_patterns.h
  src/interpreter/babel.h
  src/interpreter/blockbuf.h
  src/interpreter/blorb.h
  src/interpreter/cmd_hst.h
  src/interpreter/config.h
  src/interpreter/debugger.h
  src/interpreter/filelist.h
  src/interpreter/fizmo.h
  src/interpreter/history.h
  src/interpreter/hyphenation.h
  src/interpreter/iff.h
  src/interpreter/mathemat.h
  src/interpreter/misc.h
  src/interpreter/mt19937ar.h
  src/interpreter/object.h
  src/interpreter/output.h
  src/interpreter/property.h
  src/interpreter/routine.h
  src/interpreter/savegame.h
  src/interpreter/sound.h
  src/interpreter/stack.h
  src/interpreter/streams.h
  src/interpreter/table.h
  src/interpreter/text.h
  src/interpreter/undo.h
  src/interpreter/variable.h
  src/interpreter/wordwrap.h
  src/interpreter/zpu.h
  src/interpreter/zscii.h
  src/locales/libfizmo_locales.h
  src/locales/locale_data.h
  src/screen_interface/ScreenInterface.h
  src/screen_interface/screen_interface.h
  src/sound_interface/sound_interface.h
  src/tools/filesys.h
  src/tools/filesys_c.h
  src/tools/i18n.h
  src/tools/list.h
  src/tools/stringmap.h
  src/tools/tracelog.h
  src/tools/types.h
  src/tools/unused.h
  src/tools/z_ucs.h)

add_library(${PROJECT_NAME} ${C_FILES})
set_target_properties(${PROJECT_NAME} PROPERTIES PUBLIC_HEADER "${H_FILES}")

if (SUBPROJECT_INSTALL_PREFIX)
  set(_SUBPROJECT_INSTALL_PREFIX ${SUBPROJECT_INSTALL_PREFIX}/ )
else()
  set(_SUBPROJECT_INSTALL_PREFIX ${CMAKE_INSTALL_PREFIX}/ )
endif()

install(TARGETS ${PROJECT_NAME}
        ARCHIVE DESTINATION lib COMPONENT Libraries
        PUBLIC_HEADER DESTINATION include/fizmo COMPONENT Headers)

configure_file(src/libfizmo.pc.in ${PROJECT_NAME}.pc @ONLY)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}.pc
  DESTINATION "lib/pkgconfig")
 
#set(pc_libs_private)
#set(pc_req_private)
#
