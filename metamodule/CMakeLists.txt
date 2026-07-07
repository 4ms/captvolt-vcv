cmake_minimum_required(VERSION 3.22)

if(NOT "${METAMODULE_SDK_DIR}" STREQUAL "")
	message("METAMODULE_SDK_DIR set by CMake variable ${METAMODULE_SDK_DIR}")
elseif (DEFINED ENV{METAMODULE_SDK_DIR})
	set(METAMODULE_SDK_DIR "$ENV{METAMODULE_SDK_DIR}")
	message("METAMODULE_SDK_DIR set by environment variable ${METAMODULE_SDK_DIR}")
else()
	set(METAMODULE_SDK_DIR "${CMAKE_CURRENT_LIST_DIR}/../../metamodule-plugin-sdk")
	message("METAMODULE_SDK_DIR set to default: ${METAMODULE_SDK_DIR}")
endif()

include(${METAMODULE_SDK_DIR}/plugin.cmake)

project(CaptVoltMetaModulePlugin 
    VERSION     0.1 
    DESCRIPTION "CaptVolt Plugin for MetaModule" 
    LANGUAGES   C CXX ASM
)

add_library(CaptVolt STATIC)

set(SOURCE_DIR ${CMAKE_CURRENT_LIST_DIR}/../src)

target_sources(CaptVolt
    PRIVATE
    ${SOURCE_DIR}/filter_regs.cpp
    ${SOURCE_DIR}/plugin.cpp
    ${SOURCE_DIR}/sidofon.cpp
    ${SOURCE_DIR}/voice_regs.cpp
    ${SOURCE_DIR}/resid/dac.cc
    ${SOURCE_DIR}/resid/envelope.cc
    ${SOURCE_DIR}/resid/extfilt.cc
    ${SOURCE_DIR}/resid/filter.cc
    ${SOURCE_DIR}/resid/pot.cc
    ${SOURCE_DIR}/resid/sid.cc
    ${SOURCE_DIR}/resid/version.cc
    ${SOURCE_DIR}/resid/voice.cc
    ${SOURCE_DIR}/resid/wave.cc
)

target_include_directories(CaptVolt PRIVATE 
	${SOURCE_DIR}
	${SOURCE_DIR}/resid
)

target_compile_options(CaptVolt
	PRIVATE
	$<$<COMPILE_LANGUAGE:CXX>:-Wno-deprecated-enum-float-conversion>
)

# reSID relies on defined (wrapping) signed-integer overflow and does pointer
# type-punning. If problems arise, when running on hardware we can try enabling these flags.
# set(RESID_SAFE_FLAGS
# 	-fno-strict-aliasing
# 	-fwrapv
# 	-fno-tree-vectorize
# )
# set_source_files_properties(
# 	${SOURCE_DIR}/resid/dac.cc
# 	${SOURCE_DIR}/resid/envelope.cc
# 	${SOURCE_DIR}/resid/extfilt.cc
# 	${SOURCE_DIR}/resid/filter.cc
# 	${SOURCE_DIR}/resid/pot.cc
# 	${SOURCE_DIR}/resid/sid.cc
# 	${SOURCE_DIR}/resid/version.cc
# 	${SOURCE_DIR}/resid/voice.cc
# 	${SOURCE_DIR}/resid/wave.cc
# 	TARGET_DIRECTORY CaptVolt
# 	PROPERTIES COMPILE_OPTIONS "${RESID_SAFE_FLAGS}"
# )

target_compile_definitions(CaptVolt PRIVATE VERSION="3.4")

set_property(TARGET CaptVolt PROPERTY CXX_STANDARD 20)

if("${INSTALL_DIR}" STREQUAL "")
    set(INSTALL_DIR ${CMAKE_CURRENT_LIST_DIR}/metamodule-plugins)
endif()

create_plugin(
    SOURCE_LIB      CaptVolt                               # This is the cmake target name (defined in add_target)
    PLUGIN_NAME     captvolt                               # This must match the brand "slug" used in VCV Rack
    PLUGIN_JSON     ${CMAKE_CURRENT_LIST_DIR}/../plugin.json  # Path to the plugin.json file used by VCV Rack
    SOURCE_ASSETS   ${CMAKE_CURRENT_LIST_DIR}/assets          # Path to the assets/ dir containing the PNGs
    DESTINATION     ${INSTALL_DIR}                            # Dir where plugin .mmplugin file should be put
)
