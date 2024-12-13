cmake_minimum_required(VERSION 3.25)

# this literally doesn't compile with gcc lmao
set(CMAKE_C_COMPILER clang)
set(CMAKE_CXX_COMPILER /usr/bin/clang++)

project(htelem)

set(CMAKE_CXX_STANDARD 23)

set(HTELEM_SOURCES lib/attribute_list.h lib/braid.h lib/braided_renderer.h lib/element.h lib/html.h lib/interface_spec.h lib/render_common.h lib/static_string.h lib/util.h)

set(HTELEM_AUTOGEN_DIR "${CMAKE_SOURCE_DIR}/tools/htelem-autogen")

add_custom_command(
    OUTPUT "${HTELEM_AUTOGEN_DIR}/out/attribute_lists.def" "${HTELEM_AUTOGEN_DIR}/out/attribute_spec.def" "${HTELEM_AUTOGEN_DIR}/out/elements.def" "${HTELEM_AUTOGEN_DIR}/out/interface_spec.def"
    COMMAND "npm" ARGS "start"
    DEPENDS "${HTELEM_AUTOGEN_DIR}/index.ts"
    WORKING_DIRECTORY "${HTELEM_AUTOGEN_DIR}")

add_custom_target(ht-defs DEPENDS "${HTELEM_AUTOGEN_DIR}/out/attribute_lists.def" "${HTELEM_AUTOGEN_DIR}/out/attribute_spec.def" "${HTELEM_AUTOGEN_DIR}/out/elements.def" "${HTELEM_AUTOGEN_DIR}/out/interface_spec.def")

add_custom_target(autoformat)

function(warn_format TGT SRCS)
    add_custom_target("format-warn-${TGT}" COMMAND "clang-format" "--dry-run" "--ferror-limit=1" "${SRCS}" WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}" COMMAND_EXPAND_LISTS)
    add_dependencies(${TGT} "format-warn-${TGT}")
    add_custom_target("autoformat-${TGT}" COMMAND "clang-format" "-i" "${SRCS}" WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}" COMMAND_EXPAND_LISTS)
    add_dependencies(autoformat "autoformat-${TGT}")
endfunction()

add_library(htelem INTERFACE ${HTELEM_SOURCES})
target_include_directories(htelem INTERFACE ${CMAKE_SOURCE_DIR}/lib/)
add_dependencies(htelem ht-defs)
warn_format(htelem "${HTELEM_SOURCES}")

add_definitions(-DHTELEM_VERBOSE_TESTS)
add_subdirectory(test/)

