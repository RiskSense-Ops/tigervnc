include_directories(${FLTK_INCLUDE_DIR})
include_directories(${GETTEXT_INCLUDE_DIR})

include_directories(${CMAKE_SOURCE_DIR}/common)
set(VNCVIEWER_SOURCES
  menukey.cxx
  CConn.cxx
  DesktopWindow.cxx
  FLTKPixelBuffer.cxx
  UserDialog.cxx
  ServerDialog.cxx
  OptionsDialog.cxx
  PlatformPixelBuffer.cxx
  Viewport.cxx
  parameters.cxx
  keysym2ucs.c
  vncviewer.cxx)

if(WIN32)
  # Since vncviewer.rc is generated, local includes will be looking
  # in the wrong directory. We need to help it out.
  include_directories(${CMAKE_CURRENT_SOURCE_DIR})
  configure_file(vncviewer.rc.in vncviewer.rc)
  set(VNCVIEWER_SOURCES
    ${VNCVIEWER_SOURCES}
    ${CMAKE_CURRENT_BINARY_DIR}/vncviewer.rc)
endif()

if(WIN32)
  set(VNCVIEWER_SOURCES ${VNCVIEWER_SOURCES} win32.c)
elseif(APPLE)
  set(VNCVIEWER_SOURCES ${VNCVIEWER_SOURCES} cocoa.mm)
endif()

if(WIN32)
  set(VNCVIEWER_SOURCES ${VNCVIEWER_SOURCES} Win32PixelBuffer.cxx)
elseif(APPLE)
  set(VNCVIEWER_SOURCES ${VNCVIEWER_SOURCES} OSXPixelBuffer.cxx)
else()
  set(VNCVIEWER_SOURCES ${VNCVIEWER_SOURCES} X11PixelBuffer.cxx)
endif()

if(WIN32 AND NOT CMAKE_BUILD_TYPE STREQUAL "Debug")
  add_executable(vncviewer WIN32 ${VNCVIEWER_SOURCES})
else()
  add_executable(vncviewer ${VNCVIEWER_SOURCES})
endif()

target_link_libraries(vncviewer rfb network rdr os ${FLTK_LIBRARIES} ${GETTEXT_LIBRARIES})

if(APPLE)
  target_link_libraries(vncviewer "-framework Cocoa" "-framework Carbon")
endif()

install(TARGETS vncviewer DESTINATION ${BIN_DIR})
if(UNIX)
  install(FILES vncviewer.man DESTINATION ${MAN_DIR}/man1 RENAME vncviewer.1)
  configure_file(vncviewer.desktop.in vncviewer.desktop)
  install(FILES ${CMAKE_CURRENT_BINARY_DIR}/vncviewer.desktop DESTINATION ${DATA_DIR}/applications)
  foreach(res 16 22 24 32 48)
    install(FILES ../media/icons/tigervnc_${res}.png DESTINATION ${DATA_DIR}/icons/hicolor/${res}x${res}/apps RENAME tigervnc.png)
  endforeach()
  install(FILES ../media/icons/tigervnc.svg DESTINATION ${DATA_DIR}/icons/hicolor/scalable/apps)
endif()
