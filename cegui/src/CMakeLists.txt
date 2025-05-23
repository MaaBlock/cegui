set( CEGUI_TARGET_NAME ${CEGUI_BASE_LIBNAME} )

# we do not use cegui_gather_files() here since our needs are more complex.
file (GLOB CORE_SOURCE_FILES RELATIVE "${CMAKE_CURRENT_SOURCE_DIR}" *.cpp)
file (GLOB TEXT_SOURCE_FILES RELATIVE "${CMAKE_CURRENT_SOURCE_DIR}" text/*.cpp)
file (GLOB VIEW_SOURCE_FILES RELATIVE "${CMAKE_CURRENT_SOURCE_DIR}" views/*.cpp)
file (GLOB WIDGET_SOURCE_FILES RELATIVE "${CMAKE_CURRENT_SOURCE_DIR}" widgets/*.cpp)
file (GLOB FALAGARD_SOURCE_FILES RELATIVE "${CMAKE_CURRENT_SOURCE_DIR}" falagard/*.cpp)
file (GLOB SVG_SOURCE_FILES RELATIVE "${CMAKE_CURRENT_SOURCE_DIR}" svg/*.cpp)
set (CORE_SOURCE_FILES ${CORE_SOURCE_FILES} ${TEXT_SOURCE_FILES} ${VIEW_SOURCE_FILES} ${WIDGET_SOURCE_FILES} ${FALAGARD_SOURCE_FILES} ${SVG_SOURCE_FILES})

file (GLOB CORE_HEADER_FILES ../include/CEGUI/*.h)
file (GLOB TEXT_HEADER_FILES ../include/CEGUI/text/*.h)
file (GLOB VIEW_HEADER_FILES ../include/CEGUI/views/*.h)
file (GLOB WIDGET_HEADER_FILES ../include/CEGUI/widgets/*.h)
file (GLOB FALAGARD_HEADER_FILES ../include/CEGUI/falagard/*.h)
file (GLOB SVG_HEADER_FILES ../include/CEGUI/svg/*.h)

list (APPEND CORE_HEADER_FILES
    ${CMAKE_CURRENT_BINARY_DIR}/../include/CEGUI/Config.h
    ${CMAKE_CURRENT_BINARY_DIR}/../include/CEGUI/ModuleConfig.h
    ${CMAKE_CURRENT_BINARY_DIR}/../include/CEGUI/Version.h
)

# We don't append all to CORE_HEADER_FILES since that would mess up installation
# on systems where files are installed.
set (ALL_HEADER_FILES ${CORE_HEADER_FILES} ${TEXT_HEADER_FILES} ${VIEW_HEADER_FILES} ${WIDGET_HEADER_FILES} ${FALAGARD_HEADER_FILES} ${SVG_HEADER_FILES})

# never compile this directly, it's included in another file
list (REMOVE_ITEM CORE_SOURCE_FILES text/minibidi.cpp)

if (NOT CEGUI_USE_DEFAULT_LOGGER)
    list (REMOVE_ITEM CORE_SOURCE_FILES DefaultLogger.cpp)
endif()

if (NOT CEGUI_USE_FREETYPE)
    list (REMOVE_ITEM CORE_SOURCE_FILES text/FreeTypeFont.cpp)
endif()

if (NOT CEGUI_REGEX_MATCHER_PCRE)
    list (REMOVE_ITEM CORE_SOURCE_FILES PCRERegexMatcher.cpp)
endif()

if (NOT CEGUI_REGEX_MATCHER_STD)
    list (REMOVE_ITEM CORE_SOURCE_FILES StdRegexMatcher.cpp)
endif()

if (NOT CEGUI_BUILD_RESOURCE_PROVIDER_MINIZIP)
    list (REMOVE_ITEM CORE_SOURCE_FILES MinizipResourceProvider.cpp)
endif()

if (WIN32)
    list (REMOVE_ITEM CORE_SOURCE_FILES IconvStringTranscoder.cpp)
else()
    list (REMOVE_ITEM CORE_SOURCE_FILES Win32StringTranscoder.cpp)
endif()

if (NOT WIN32)
    list (REMOVE_ITEM CORE_SOURCE_FILES Win32ClipboardProvider.cpp)
endif()

if (NOT ANDROID)
    list (REMOVE_ITEM CORE_SOURCE_FILES AndroidUtils.cpp)
    list (REMOVE_ITEM CORE_HEADER_FILES AndroidUtils.h)
endif()

# we do not use the common header install function since we need to install to
# multiple sub-dirs.
cegui_add_library_impl(${CEGUI_TARGET_NAME} FALSE CORE_SOURCE_FILES ALL_HEADER_FILES TRUE FALSE)
if (ANDROID)
    target_link_libraries (${CEGUI_TARGET_NAME} log)
endif ()

source_group("Source Files\\text" FILES ${TEXT_SOURCE_FILES})
source_group("Source Files\\view" FILES ${VIEW_SOURCE_FILES})
source_group("Source Files\\widget" FILES ${WIDGET_SOURCE_FILES})
source_group("Source Files\\falagard" FILES ${FALAGARD_SOURCE_FILES})
source_group("Source Files\\svg" FILES ${SVG_SOURCE_FILES})

source_group("Header Files\\text" FILES ${TEXT_HEADER_FILES})
source_group("Header Files\\view" FILES ${VIEW_HEADER_FILES})
source_group("Header Files\\widget" FILES ${WIDGET_HEADER_FILES})
source_group("Header Files\\falagard" FILES ${FALAGARD_HEADER_FILES})
source_group("Header Files\\svg" FILES ${SVG_HEADER_FILES})

if (NOT WIN32 AND NOT ANDROID)
    cegui_add_dependency(${CEGUI_TARGET_NAME} ICONV)
endif()

if (CEGUI_REGEX_MATCHER_PCRE)
    cegui_add_dependency(${CEGUI_TARGET_NAME} PCRE)
endif()

if (CEGUI_USE_RAQM)
    cegui_add_dependency(${CEGUI_TARGET_NAME} FREETYPE)
    cegui_add_dependency(${CEGUI_TARGET_NAME} FRIBIDI)
    cegui_add_dependency(${CEGUI_TARGET_NAME} HARFBUZZ)
    cegui_add_dependency(${CEGUI_TARGET_NAME} RAQM)
else()
    if (CEGUI_USE_FREETYPE)
        cegui_add_dependency(${CEGUI_TARGET_NAME} FREETYPE)

        include(CheckSymbolExists)
        set (CMAKE_REQUIRED_INCLUDES ${CMAKE_REQUIRED_INCLUDES} ${FREETYPE_INCLUDE_DIRS})
        set (CMAKE_REQUIRED_LIBRARIES ${CMAKE_REQUIRED_LIBRARIES} ${FREETYPE_LIBRARIES})
        check_symbol_exists(FT_CONFIG_OPTION_USE_HARFBUZZ "freetype/freetype.h" FREETYPE_USES_HARFBUZZ)
        if (FREETYPE_USES_HARFBUZZ)
            cegui_add_dependency(${CEGUI_TARGET_NAME} HARFBUZZ)
        endif ()
    endif ()

    if (CEGUI_USE_FRIBIDI)
        cegui_add_dependency(${CEGUI_TARGET_NAME} FRIBIDI)
    endif ()
endif ()

if (CEGUI_BUILD_RESOURCE_PROVIDER_MINIZIP)
    target_link_libraries(${CEGUI_BASE_LIBNAME} unofficial::minizip::minizip)
endif()

#if (CEGUI_BUILD_RESOURCE_PROVIDER_MINIZIP)
#    cegui_add_dependency(${CEGUI_TARGET_NAME} MINIZIP)
#    if (MINGW)
#        target_link_libraries(${CEGUI_TARGET_NAME} shlwapi)
#    endif ()
#endif ()

if (WIN32 AND NOT MINGW)
    cegui_target_link_libraries(${CEGUI_TARGET_NAME} winmm debug DbgHelp)
elseif (UNIX AND NOT APPLE AND NOT ANDROID)
    # This is intentionally not using 'cegui_target_link_libraries'
    target_link_libraries(${CEGUI_TARGET_NAME} ${CMAKE_DL_LIBS})
elseif (MINGW)
    cegui_target_link_libraries(${CEGUI_TARGET_NAME} ${CMAKE_DL_LIBS})
endif()

if (APPLE AND CEGUI_BUILD_SHARED_LIBS_WITH_STATIC_DEPENDENCIES)
    set_property(TARGET ${CEGUI_TARGET_NAME} APPEND PROPERTY LINK_FLAGS "-framework Carbon")
endif()

install(FILES ${CORE_HEADER_FILES} DESTINATION ${CEGUI_INSTALL_INCLUDE_DIR}/CEGUI COMPONENT cegui_devel)
install(FILES ${TEXT_HEADER_FILES} DESTINATION ${CEGUI_INSTALL_INCLUDE_DIR}/CEGUI/text COMPONENT cegui_devel)
install(FILES ${VIEW_HEADER_FILES} DESTINATION ${CEGUI_INSTALL_INCLUDE_DIR}/CEGUI/views COMPONENT cegui_devel)
install(FILES ${WIDGET_HEADER_FILES} DESTINATION ${CEGUI_INSTALL_INCLUDE_DIR}/CEGUI/widgets COMPONENT cegui_devel)
install(FILES ${FALAGARD_HEADER_FILES} DESTINATION ${CEGUI_INSTALL_INCLUDE_DIR}/CEGUI/falagard COMPONENT cegui_devel)
install(FILES ${SVG_HEADER_FILES} DESTINATION ${CEGUI_INSTALL_INCLUDE_DIR}/CEGUI/svg COMPONENT cegui_devel)

