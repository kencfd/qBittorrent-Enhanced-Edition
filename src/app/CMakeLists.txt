project(qbt_executable)
include_directories(${CMAKE_CURRENT_BINARY_DIR})

set(QBT_APP_HEADERS
application.h
filelogger.h
)

set(QBT_APP_SOURCES
application.cpp
filelogger.cpp
main.cpp
)

# translations
file(GLOB QBT_TS_FILES ../lang/*.ts)
get_filename_component(QBT_QM_FILES_BINARY_DIR "${CMAKE_CURRENT_BINARY_DIR}/../lang" ABSOLUTE)
set_source_files_properties(${QBT_TS_FILES} PROPERTIES OUTPUT_LOCATION "${QBT_QM_FILES_BINARY_DIR}")

if (QT5)
    find_package(Qt5 COMPONENTS LinguistTools REQUIRED)
    qt5_add_translation(QBT_QM_FILES ${QBT_TS_FILES})
else (QT5)
    qt4_add_translation(QBT_QM_FILES ${QBT_TS_FILES})
endif (QT5)

get_filename_component(_lang_qrc_src "${CMAKE_CURRENT_SOURCE_DIR}/../lang.qrc" ABSOLUTE)
get_filename_component(_lang_qrc_dst "${CMAKE_CURRENT_BINARY_DIR}/../lang.qrc" ABSOLUTE)
get_filename_component(_lang_qrc_dst_dir "${CMAKE_CURRENT_BINARY_DIR}/../" ABSOLUTE)

message(STATUS "copying ${_lang_qrc_src} -> ${_lang_qrc_dst}")
file(COPY ${_lang_qrc_src} DESTINATION ${_lang_qrc_dst_dir})

set_source_files_properties("${_lang_qrc_dst}" PROPERTIES GENERATED True)
foreach(qm_file ${QBT_QM_FILES})
    set_source_files_properties("${_lang_qrc_dst}" PROPERTIES OBJECT_DEPENDS ${qm_file})
endforeach()

set(QBT_APP_RESOURCES
../icons.qrc
../searchengine.qrc
"${_lang_qrc_dst}"
)

# With AUTORCC rcc is ran by cmake before language files are generated,
# and thus we call rcc explicitly
if (QT5)
    qt5_add_resources(QBT_APP_RESOURCE_SOURCE ${QBT_APP_RESOURCES})
else (QT5)
    qt4_add_resources(QBT_APP_RESOURCE_SOURCE ${QBT_APP_RESOURCES})
endif (QT5)

if (WIN32)
    if (MINGW)
        list (APPEND QBT_APP_SOURCES ../qbittorrent_mingw.rc)
    else (MINGW)
        list (APPEND QBT_APP_SOURCES ../qbittorrent.rc)
    endif (MINGW)
    list(APPEND QBT_APP_SOURCES ../qbittorrent.exe.manifest)
endif (WIN32)

if (UNIX)
    list(APPEND QBT_APP_HEADERS stacktrace.h)
endif (UNIX)

if (STACKTRACE_WIN)
    list(APPEND QBT_APP_HEADERS stacktrace_win.h)
    if (GUI)
        list(APPEND QBT_APP_HEADERS stacktrace_win_dlg.h)
    endif (GUI)
endif (STACKTRACE_WIN)

# usesystemqtsingleapplication {
#     nogui {
#         CONFIG += qtsinglecoreapplication
#     } else {
#         CONFIG += qtsingleapplication
#     }
# } else {
#     nogui {
#         include(qtsingleapplication/qtsinglecoreapplication.pri)
#     } else {
#         include(qtsingleapplication/qtsingleapplication.pri)
#     }
# }

# upgrade code
list(APPEND QBT_APP_HEADERS upgrade.h)
list(APPEND QBT_TARGET_LIBRARIES qbt_base)

if (GUI)
    list(APPEND QBT_TARGET_LIBRARIES qbt_searchengine qbt_gui)
    include_directories(../gui
        ${CMAKE_CURRENT_BINARY_DIR}/../gui
    )
endif (GUI)

if (WEBUI)
    list(APPEND QBT_TARGET_LIBRARIES qbt_webui)
endif (WEBUI)

# we have to include resources into the bundle
if (APPLE)
    set(OSX_RES_SRC_DIR "${qBittorrent_SOURCE_DIR}/dist/mac")
    list(APPEND QBT_APP_RESOURCE_SOURCE
        "${OSX_RES_SRC_DIR}/qt.conf"
        "${OSX_RES_SRC_DIR}/qBitTorrentDocument.icns"
        "${OSX_RES_SRC_DIR}/qbittorrent_mac.icns")
    set_source_files_properties(
        "${OSX_RES_SRC_DIR}/qt.conf"
        "${OSX_RES_SRC_DIR}/qBitTorrentDocument.icns"
        "${OSX_RES_SRC_DIR}/qbittorrent_mac.icns"
        PROPERTIES
            MACOSX_PACKAGE_LOCATION Resources)
    set(QT_TR_DIR "${qBittorrent_SOURCE_DIR}/dist/qt-translations")
    set(QT_TRANSLATIONS
        ${QT_TR_DIR}/qt_ar.qm
        ${QT_TR_DIR}/qt_bg.qm
        ${QT_TR_DIR}/qt_ca.qm
        ${QT_TR_DIR}/qt_cs.qm
        ${QT_TR_DIR}/qt_da.qm
        ${QT_TR_DIR}/qt_de.qm
        ${QT_TR_DIR}/qt_es.qm
        ${QT_TR_DIR}/qt_eu.qm
        ${QT_TR_DIR}/qt_fi.qm
        ${QT_TR_DIR}/qt_fr.qm
        ${QT_TR_DIR}/qt_gl.qm
        ${QT_TR_DIR}/qt_he.qm
        ${QT_TR_DIR}/qt_hu.qm
        ${QT_TR_DIR}/qt_it.qm
        ${QT_TR_DIR}/qt_ja.qm
        ${QT_TR_DIR}/qt_ko.qm
        ${QT_TR_DIR}/qt_lt.qm
        ${QT_TR_DIR}/qt_nl.qm
        ${QT_TR_DIR}/qt_pl.qm
        ${QT_TR_DIR}/qt_pt.qm
        ${QT_TR_DIR}/qt_pt_BR.qm
        ${QT_TR_DIR}/qt_ru.qm
        ${QT_TR_DIR}/qt_sk.qm
        ${QT_TR_DIR}/qt_sv.qm
        ${QT_TR_DIR}/qt_tr.qm
        ${QT_TR_DIR}/qt_uk.qm
        ${QT_TR_DIR}/qt_zh_CN.qm
        ${QT_TR_DIR}/qt_zh_TW.qm
    )
    list(APPEND QBT_APP_RESOURCE_SOURCE ${QT_TRANSLATIONS})
    set_source_files_properties(${QT_TRANSLATIONS}
        PROPERTIES MACOSX_PACKAGE_LOCATION translations)
endif (APPLE)

add_executable(${QBT_TARGET_NAME} ${QBT_APP_HEADERS} ${QBT_APP_SOURCES} ${QBT_QM_FILES} ${QBT_APP_RESOURCE_SOURCE})
set_target_properties(${QBT_TARGET_NAME}
    PROPERTIES
        AUTOUIC True
        AUTORCC True
        MACOSX_BUNDLE True
)

if (GUI AND WIN32)
    set_target_properties(${QBT_TARGET_NAME} PROPERTIES WIN32_EXECUTABLE True)
endif (GUI AND WIN32)

target_link_libraries(${QBT_TARGET_NAME} ${QBT_TARGET_LIBRARIES} QtSingleApplication::QtSingleApplication)

if (APPLE)
    set(qbt_BUNDLE_NAME "${QBT_TARGET_NAME}")

# substitute @EXECUTABLE@ in dist/mac/Info.plist
    set(EXECUTABLE ${qbt_BUNDLE_NAME})
    configure_file(${qBittorrent_SOURCE_DIR}/dist/mac/Info.plist ${qBittorrent_BINARY_DIR}/dist/mac/Info.plist @ONLY)

    set_target_properties(${QBT_TARGET_NAME} PROPERTIES
        MACOSX_BUNDLE_BUNDLE_NAME "${qbt_BUNDLE_NAME}"
        MACOSX_BUNDLE_INFO_PLIST ${qBittorrent_BINARY_DIR}/dist/mac/Info.plist
    )
endif (APPLE)

# installation
install(TARGETS ${QBT_TARGET_NAME}
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    BUNDLE  DESTINATION .
    COMPONENT runtime)

if (APPLE AND GUI)
    install(SCRIPT ${OSX_RES_SRC_DIR}/bundle.cmake)
endif (APPLE AND GUI)
