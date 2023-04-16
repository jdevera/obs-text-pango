cmake_minimum_required(VERSION 3.19...3.26)

project(obs-text-pango VERSION 1.0.1)
add_library(${CMAKE_PROJECT_NAME} MODULE)

# The name (yours or your organization's) you want to see as the
# author of the plugin (in the plugin's metadata itself and in the installers)
set(PLUGIN_AUTHOR "Kurt Kartaltepe")

# Replace 'https://www.example.com` with a link to the website of your plugin or repository
set(PLUGIN_WEBSITE "https://github.com/kkartaltepe/obs-text-pango")

# Unique Bundle ID for macOS releases (used both in
# the installer and when submitting the installer for notarization)
set(MACOS_BUNDLEID "com.example.${CMAKE_PROJECT_NAME}")

# The maintainer email address you want to put in Linux packages
set(LINUX_MAINTAINER_EMAIL "noreply@example.com")

# Add your custom source files here - header files are optional and only required for visibility
# e.g. in Xcode or Visual Studio
target_sources(${CMAKE_PROJECT_NAME} PRIVATE
	src/text-pango.c
	src/text-pango.h
	src/text-utilities.h
    )

# Set this repo's CMake goodies in the module path for CMake
set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${CMAKE_SOURCE_DIR}/cmake/")

include(ObsPluginHelpers)

set(CMAKE_C_STANDARD 11)

# include(ObsHelpers)

# Find and setup dependencies
# ---------------------------

# Import libobs as main plugin dependency
find_package(libobs REQUIRED)

find_package(Pango REQUIRED)
find_package(Cairo REQUIRED)
find_package(Pangocairo REQUIRED)
find_package(Pangoft2 REQUIRED)
if(WIN32)
	find_package(Pangowin32 REQUIRED)
endif()
find_package(Fontconfig REQUIRED)
find_package(Freetype REQUIRED)


include_directories(
	${OBS_INCLUDE_DIRS}
	${PANGO_INCLUDE_DIRS}
	${CAIRO_INCLUDE_DIRS}
	${PANGOCAIRO_INCLUDE_DIRS}
	${PANGOFT2_INCLUDE_DIRS}
	${FONTCONFIG_INCLUDE_DIRS}
	${FREETYPE_INCLUDE_DIRS}
    )

if(NOT WIN32)
	FIND_PACKAGE(PkgConfig)
	PKG_CHECK_MODULES(GLIB glib-2.0 REQUIRED)
	include_directories(${GLIB_INCLUDE_DIRS})
endif()

target_link_directories(
    ${CMAKE_PROJECT_NAME}
    PRIVATE
	${FREETYPE_LIBRARY_DIRS}
	${PANGO_LIBRARY_DIRS}
	${CAIRO_LIBRARY_DIRS}
	${FONTCONFIG_LIBRARY_DIRS}
    )

set(win32_static_LIBRARIES
	intl
	mini-fribidi
	pixman-1_static
	zlib
	harfbuzz
	libpng16_static
	expat
	usp10)
if(WIN32)
	set(PLATFORM_LIBRARIES ${win32_static_LIBRARIES})
endif()

target_link_libraries(${CMAKE_PROJECT_NAME}
    PRIVATE
    OBS::libobs
	${OBS_LIBRARIES}
	${CAIRO_LIBRARIES}
	${PANGO_LIBRARIES}
	${PANGOWIN32_LIBRARIES}
	${PANGOCAIRO_LIBRARIES}
	${PANGOFT2_LIBRARIES}
	${FREETYPE_LIBRARIES}
	${FONTCONFIG_LIBRARIES}
	${PLATFORM_LIBRARIES}
    ) # win only currently

target_compile_definitions(${CMAKE_PROJECT_NAME} PRIVATE ${CAIRO_C_FLAGS})

# ---------------------------------

# /!\ TAKE NOTE: No need to edit things past this point /!\

# --- Platform-independent build settings ---

target_include_directories(${CMAKE_PROJECT_NAME} PRIVATE ${CMAKE_SOURCE_DIR}/src)

#target_link_libraries(${CMAKE_PROJECT_NAME} PRIVATE OBS::libobs)

# --- End of section ---

# --- Windows-specific build settings and tasks ---
if(OS_WINDOWS)
  configure_file(cmake/bundle/windows/installer-Windows.iss.in
                 ${CMAKE_BINARY_DIR}/installer-Windows.generated.iss)

  configure_file(cmake/bundle/windows/resource.rc.in ${CMAKE_BINARY_DIR}/${CMAKE_PROJECT_NAME}.rc)
  target_sources(${CMAKE_PROJECT_NAME} PRIVATE ${CMAKE_BINARY_DIR}/${CMAKE_PROJECT_NAME}.rc)

  if(MSVC)
    target_compile_options(${CMAKE_PROJECT_NAME} PRIVATE /W4)
  endif()
  # --- End of section ---

  # -- macOS specific build settings and tasks --
elseif(OS_MACOS)
  configure_file(cmake/bundle/macos/installer-macos.pkgproj.in
                 ${CMAKE_BINARY_DIR}/installer-macos.generated.pkgproj)

  set(MACOSX_PLUGIN_GUI_IDENTIFIER "${MACOS_BUNDLEID}")
  set(MACOSX_PLUGIN_BUNDLE_VERSION "${CMAKE_PROJECT_VERSION}")
  set(MACOSX_PLUGIN_SHORT_VERSION_STRING "1")

  target_compile_options(${CMAKE_PROJECT_NAME} PRIVATE -Wall)
  # --- End of section ---

  # --- Linux-specific build settings and tasks ---
else()
  target_compile_options(${CMAKE_PROJECT_NAME} PRIVATE -Wall)
endif()
# --- End of section ---

setup_plugin_target(${CMAKE_PROJECT_NAME})
