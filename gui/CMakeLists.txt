 # BEGIN_COMMON_COPYRIGHT_HEADER
 # (c)LGPL2+
 #
 #
 # Copyright: 2012-2013 Boomaga team https://github.com/Boomaga
 # Authors:
 #   Alexander Sokoloff <sokoloff.a@gmail.com>
 #
 # This program or library is free software; you can redistribute it
 # and/or modify it under the terms of the GNU Lesser General Public
 # License as published by the Free Software Foundation; either
 # version 2.1 of the License, or (at your option) any later version.
 #
 # This library is distributed in the hope that it will be useful,
 # but WITHOUT ANY WARRANTY; without even the implied warranty of
 # MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
 # Lesser General Public License for more details.
 #
 # You should have received a copy of the GNU Lesser General
 # Public License along with this library; if not, write to the
 # Free Software Foundation, Inc., 51 Franklin Street, Fifth Floor,
 # Boston, MA 02110-1301 USA
 #
 # END_COMMON_COPYRIGHT_HEADER


cmake_minimum_required( VERSION 2.6 )

project(boomagaGui)

set(HEADERS
    kernel/sheet.h
    kernel/inputfile.h
    kernel/tmppdffile.h
    kernel/layout.h
    kernel/render.h
    kernel/project.h
    kernel/printer.h
    boomagatypes.h
    mainwindow.h
    settings.h
    widgets.h
    dbus.h
    printersettings/printersettings.h
    previewwidget.h
    aboutdialog/aboutdialog.h
    translations/translatorsinfo/translatorsinfo.h
)

set(SOURCES
    main.cpp
    kernel/sheet.cpp
    kernel/inputfile.cpp
    kernel/tmppdffile.cpp
    kernel/layout.cpp
    kernel/render.cpp
    kernel/project.cpp
    kernel/printer.cpp
    boomagatypes.cpp
    mainwindow.cpp
    settings.cpp
    widgets.cpp
    dbus.cpp
    printersettings/printersettings.cpp
    previewwidget.cpp
    aboutdialog/aboutdialog.cpp
    translations/translatorsinfo/translatorsinfo.cpp
)

set (MOCS
    kernel/project.h
    kernel/tmppdffile.h
    kernel/render.h

    settings.h
    mainwindow.h
    widgets.h
    dbus.h
    printersettings/printersettings.h
    previewwidget.h
    aboutdialog/aboutdialog.h
)

set(LIBRARIES
)

set(QT_USE_QTDBUS 1)

set(UI_FILES
    mainwindow.ui
    printersettings/printersettings.ui
    aboutdialog/aboutdialog.ui
)

# QRC files .................
set(RESOURCES
    resource.qrc
)

# add translations ...
file (GLOB TS_FILES
    translations/boomaga*.ts
)

set(TRANSLATIONS_DIR
    "${GUI_SHARE_DIR}/translations"
)
add_definitions(-DTRANSLATIONS_DIR=\"${TRANSLATIONS_DIR}\")


############################################
if (NOT CMAKE_BUILD_TYPE)
    set ( CMAKE_BUILD_TYPE Release )
endif (NOT CMAKE_BUILD_TYPE)
#set(CMAKE_BUILD_TYPE Debug)

if(CMAKE_COMPILER_IS_GNUCXX)
    add_definitions (-Wall -pedantic)
endif()

#*******************************************

SET(CMAKE_MODULE_PATH ${PROJECT_SOURCE_DIR}/cmake)
find_package(Qt4 REQUIRED QUIET)
find_package(Cups REQUIRED QUIET)
find_package(snappy REQUIRED QUIET)
find_package(Poppler REQUIRED QUIET)

include_directories(
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_CURRENT_SOURCE_DIR}/third-party
    ${CMAKE_CURRENT_SOURCE_DIR}/third-party/libspectre
    ${CMAKE_SOURCE_DIR}
    ${CMAKE_CURRENT_BINARY_DIR}
    ${CUPS_INCLUDE_DIR}
    ${SNAPPY_INCLUDE_DIRS}
    ${POPPLER_INCLUDE_DIRS}
    ${POPPLERQT_INCLUDE_DIRS}
)

include(${QT_USE_FILE})
qt4_wrap_cpp(MOC_SOURCES ${MOCS})
qt4_add_resources(QRC_SOURCES ${RESOURCES})
qt4_wrap_ui(UI_HEADERS ${UI_FILES})
qt4_add_translation(QM_FILES ${TS_FILES})

include(translations/translatorsinfo/CMakeLists.txt)
get_translatorsinfo_qrc(translatorsinfo_qrc)
qt4_add_resources(QRC_CXX_SOURCES ${translatorsinfo_qrc})

add_executable(${INSTANCE_NAME} ${HEADERS} ${SOURCES} ${MOC_SOURCES} ${QM_FILES} ${QRC_SOURCES} ${UI_HEADERS} ${QRC_CXX_SOURCES})
target_link_libraries(${INSTANCE_NAME} ${LIBRARIES} ${QT_LIBRARIES} ${CUPS_LIBRARIES} ${SNAPPY_LIBRARIES} ${POPPLER_LIBRARIES} ${POPPLERQT_LIBRARIES})

install(TARGETS ${INSTANCE_NAME} RUNTIME DESTINATION ${GUI_DIR})
install(FILES ${QM_FILES} DESTINATION ${TRANSLATIONS_DIR})

configure_file(${CMAKE_CURRENT_SOURCE_DIR}/org.boomaga.service
               ${CMAKE_CURRENT_BINARY_DIR}/org.boomaga.service
               @ONLY)

install(FILES ${CMAKE_CURRENT_BINARY_DIR}/org.boomaga.service DESTINATION ${DBUS_SERVICE_DIR})


add_subdirectory(pdfmerger)
