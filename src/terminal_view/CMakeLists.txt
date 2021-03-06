find_package(Freetype REQUIRED)
find_package(OpenGL REQUIRED)
find_package(Qt5 COMPONENTS Gui REQUIRED)  # apt install qtbase5-dev libqt5gui5

if(APPLE)
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(fontconfig REQUIRED IMPORTED_TARGET fontconfig)
    pkg_check_modules(harfbuzz REQUIRED IMPORTED_TARGET harfbuzz)
elseif("${CMAKE_SYSTEM}" MATCHES "Linux")
      #find_package(fontconfig)
    find_package(PkgConfig REQUIRED)
    pkg_check_modules(harfbuzz REQUIRED IMPORTED_TARGET harfbuzz)
elseif("${CMAKE_SYSTEM}" MATCHES "Windows")
    # installed via vcpkg
    #find_package(unofficial-fontconfig)
    find_package(harfbuzz CONFIG REQUIRED)
endif()

add_library(terminal_view STATIC
    CellBackground.cpp CellBackground.h
    FontManager.cpp FontManager.h
    GLCursor.cpp GLCursor.h
    GLTextShaper.cpp GLTextShaper.h
    GLRenderer.cpp GLRenderer.h
    TerminalView.cpp TerminalView.h
)

option(LIBTERMINAL_VIEW_FONT_RENDER_CACHE "Enables internal render cache [default: ON]." ON)
if(LIBTERMINAL_VIEW_FONT_RENDER_CACHE)
    target_compile_definitions(terminal_view PUBLIC LIBTERMINAL_VIEW_FONT_RENDER_CACHE=1)
endif()

target_include_directories(terminal_view PUBLIC ${PROJECT_SOURCE_DIR}/src ${CMAKE_SOURCE_DIR}/src)

set(TERMINAL_VIEW_LIBRARIES Qt5::Gui Freetype::Freetype terminal)
if(APPLE)
    list(APPEND TERMINAL_VIEW_LIBRARIES PkgConfig::fontconfig)
    list(APPEND TERMINAL_VIEW_LIBRARIES PkgConfig::harfbuzz)
elseif("${CMAKE_SYSTEM}" MATCHES "Linux" OR APPLE)
    list(APPEND TERMINAL_VIEW_LIBRARIES fontconfig)
    list(APPEND TERMINAL_VIEW_LIBRARIES PkgConfig::harfbuzz)
elseif("${CMAKE_SYSTEM}" MATCHES "Windows")
    list(APPEND TERMINAL_VIEW_LIBRARIES harfbuzz::harfbuzz)
    #list(APPEND GLTERM_LIBRARIES unofficial::fontconfig::fontconfig)
endif()

message(STATUS "[libterminal_view] Font Render Cache: ${LIBTERMINAL_VIEW_FONT_RENDER_CACHE}")

target_link_libraries(terminal_view PUBLIC ${TERMINAL_VIEW_LIBRARIES})
