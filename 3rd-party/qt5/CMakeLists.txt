#
#    Copyright 2012 Kai Pastor
#    
#    This file is part of OpenOrienteering.
# 
#    OpenOrienteering is free software: you can redistribute it and/or modify
#    it under the terms of the GNU General Public License as published by
#    the Free Software Foundation, either version 3 of the License, or
#    (at your option) any later version.
# 
#    OpenOrienteering is distributed in the hope that it will be useful,
#    but WITHOUT ANY WARRANTY; without even the implied warranty of
#    MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the
#    GNU General Public License for more details.
# 
#    You should have received a copy of the GNU General Public License
#    along with OpenOrienteering.  If not, see <http://www.gnu.org/licenses/>.


project(Qt5)

cmake_minimum_required(VERSION 2.8.3)
 
include(ExternalProject)

# Configuration options

if(NOT QT5_VERSION)
	set(QT5_VERSION 5.0.0 CACHE STRING
	  "Version number of the Qt5 library, minumum recommended value: 5.0.0")
endif(NOT QT5_VERSION)
mark_as_advanced(QT5_VERSION)

if(QT5_VERSION STREQUAL 5.0.0)
	set(QT5_MD5 1a1b73f913a965f6b0cf02bb820c8a03)
endif()

if("${CMAKE_GENERATOR}" MATCHES "Makefiles" OR 
   "${CMAKE_EXTRA_GENERATOR}" MATCHES "Makefiles")
	# use magic MAKE macro
	set(QT5_MAKE_COMMAND "\$(MAKE)" CACHE STRING
	  "The 'make' command for building the Qt library")
else()
	# require make
	set(QT5_MAKE_COMMAND "${SH_PROGRAM}" -l -e "${CMAKE_CURRENT_BINARY_DIR}/qt5-make" make CACHE STRING
	  "The 'make' command for building the Qt library")
endif()
mark_as_advanced(QT5_MAKE_COMMAND)

find_program(SH_PROGRAM sh 
  PATHS 
  "[HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Uninstall\\MSYS-1.0_is1;Inno Setup: App Path]/bin"
  "[HKEY_LOCAL_MACHINE\\SOFTWARE\\Microsoft\\Windows\\CurrentVersion\\Uninstall\\MinGW;InstallLocation]/bin"
  C:
  C:/MinGW
  PATH_SUFFIXES msys/1.0/bin)
mark_as_advanced(SH_PROGRAM)

# Build definition

# Don't touch the configured qt5-config unless the content really changes.
add_custom_command(OUTPUT qt5-config
  COMMAND ${CMAKE_COMMAND} -E copy_if_different qt5-config.new qt5-config
  DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/qt5-config.new
)

ExternalProject_Add(Qt5
  DOWNLOAD_DIR ${PROJECT_SOURCE_DIR}/download
  URL "http://releases.qt-project.org/qt5/${QT5_VERSION}/single/qt-everywhere-opensource-src-${QT5_VERSION}.tar.gz"
  URL_MD5 ${QT5_MD5}
  CONFIGURE_COMMAND "${SH_PROGRAM}" -l -e ${CMAKE_CURRENT_BINARY_DIR}/qt5-config
  BUILD_COMMAND ${QT5_MAKE_COMMAND}
  INSTALL_COMMAND ${QT5_MAKE_COMMAND} install
)
ExternalProject_Get_Property(Qt5 SOURCE_DIR)
ExternalProject_Get_Property(Qt5 BINARY_DIR)
ExternalProject_Get_Property(Qt5 INSTALL_DIR)

# Trigger configure when qt5-config has changed.
ExternalProject_Add_Step(Qt5 configure-dependencies
  COMMAND ${CMAKE_COMMAND} -E echo_append "" # Do nothing
  DEPENDEES update patch
  DEPENDERS configure
  DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/qt5-config
)

# Exported configuration

if (CMAKE_CROSSCOMPILING)
	set(QT5_DIR "${INSTALL_DIR}/${GNU_SYSTEM_NAME}" CACHE INTERNAL "The Qt5 target platform directory." FORCE)
else()
	set(QT5_DIR "${INSTALL_DIR}" CACHE INTERNAL "The Qt5 target platform directory." FORCE)
endif()
set(QT5_HOST_DIR "${INSTALL_DIR}" CACHE INTERNAL "The Qt5 host platform directory." FORCE)
set(QT_TRANSLATIONS_DIR "${QT5_DIR}/translations" CACHE INTERNAL "The Qt5 translation directory." FORCE)

# Build script configuration

configure_file(qt5-config.in qt5-config.new @ONLY)
configure_file(qt5-make.in qt5-make @ONLY)