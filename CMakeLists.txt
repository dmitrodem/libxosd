cmake_minimum_required(VERSION 3.0)

project(xosd VERSION 2.2.14)

option(USE_XINERAMA "Use Xinerama extension" ON)
option(USE_XRANDR   "Use Xrandr extension"   ON)

set(XOSD_LDFLAGS "")

include(FindThreads REQUIRED)
list(APPEND XOSD_LDFLAGS ${CMAKE_THREAD_LIBS_INIT})

find_package(PkgConfig REQUIRED)
pkg_check_modules(X11 REQUIRED x11)
pkg_check_modules(XEXT REQUIRED xext)
list(APPEND XOSD_LDFLAGS ${X11_LDFLAGS} ${XEXT_LDFLAGS})

if(USE_XRANDR)
    pkg_check_modules(XRANDR REQUIRED xrandr>=1.5)
    if(XRANDR_FOUND)
        add_definitions(-DHAS_XRANDR=1)
        list(APPEND XOSD_LDFLAGS ${XRANDR_LDFLAGS})
    endif()
endif()

if(USE_XINERAMA)
    pkg_check_modules(XINERAMA REQUIRED xinerama)
    if(XINERAMA_FOUND)
        add_definitions(-DHAS_XINERAMA=1)
        list(APPEND XOSD_LDFLAGS ${XINERAMA_LDFLAGS})
    endif()
endif()

add_library(xosd SHARED src/libxosd/xosd.c)
set_target_properties(xosd PROPERTIES VERSION ${PROJECT_VERSION})
add_executable(osd_cat src/osd_cat.c)
target_link_libraries(osd_cat xosd ${XOSD_LDFLAGS})
configure_file("xosd.pc.in" "xosd.pc" @ONLY)


install(TARGETS xosd DESTINATION lib)
install(TARGETS osd_cat DESTINATION bin)
install(FILES "${CMAKE_CURRENT_SOURCE_DIR}/src/xosd.h" DESTINATION include)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/xosd.pc" DESTINATION lib/pkgconfig)
set(MAN_PAGES 
    "xosd_set_bar_length.3"
    "xosd_set_pos.3"
    "osd_cat.1"
    "xosd_is_onscreen.3"
    "xosd_hide.3"
    "xosd.3"
    "xosd_uninit.3"
    "xosd-config.1"
    "xosd_destroy.3"
    "xosd_set_align.3"
    "xosd_set_shadow_offset.3"
    "xosd_display.3"
    "xosd_set_vertical_offset.3"
    "xosd_get_number_lines.3"
    "xosd_set_horizontal_offset.3"
    "xosd_create.3"
    "xosd_show.3")
foreach(MAN ${MAN_PAGES})
    install(FILES "${CMAKE_CURRENT_SOURCE_DIR}/man/${MAN}" DESTINATION man)
endforeach()
