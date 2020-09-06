cmake_minimum_required(VERSION 3.2)

project(MeetingRecorder VERSION 0.0.1)
include(ExternalProject)
include(GNUInstallDirs)
include(CheckCXXCompilerFlag)

# Prepare gobal CMAKE configuration specific to the current project
set(SDK_BUILD_DIR "${CMAKE_BINARY_DIR}/WORK")       # SDK build in WORK. Keep all in it.
set(LINPHONE_OUTPUT_DIR "${CMAKE_BINARY_DIR}/linphone-sdk/desktop")

set(APPLICATION_OUTPUT_DIR "${CMAKE_BINARY_DIR}/OUTPUT")

set(CMAKE_PREFIX_PATH "${LINPHONE_OUTPUT_DIR};${APPLICATION_OUTPUT_DIR}${PREFIX_PATH}")
string(REPLACE ";" "|" PREFIX_PATH "${CMAKE_PREFIX_PATH}")

if(WIN32 OR APPLE)
else()
	option(ENABLE_V4L "Ability to capture and display video using libv4l2 or libv4l." YES)
endif()

option(ENABLE_RELATIVE_PREFIX "Set Internal packages relative to the binary" YES)


set(LINPHONE_OUTPUT_DIR "${CMAKE_BINARY_DIR}/linphone-sdk/desktop")
set(SDK_BUILD_DIR "${CMAKE_BINARY_DIR}/WORK")

ExternalProject_Add(sdk PREFIX "${CMAKE_BINARY_DIR}/sdk"
    SOURCE_DIR "${CMAKE_SOURCE_DIR}/linphone-sdk"
    INSTALL_DIR "${LINPHONE_OUTPUT_DIR}"
    STAMP_DIR "${SDK_BUILD_DIR}/stamp"
    BINARY_DIR "${SDK_BUILD_DIR}"
    STEP_TARGETS build
    BUILD_COMMAND ${CMAKE_COMMAND} --build <BINARY_DIR> --config $<CONFIG> ${PROJECT_BUILD_COMMAND}
    INSTALL_COMMAND ${CMAKE_COMMAND} -E echo "Install step is already done at build time."
    LIST_SEPARATOR |
    CMAKE_ARGS ${APP_OPTIONS} ${USER_ARGS} -DCMAKE_INSTALL_PREFIX:PATH=<INSTALL_DIR> -DCMAKE_PREFIX_PATH=${PREFIX_PATH}
)

include(FindPkgConfig)

set(APP_DEPENDS sdk)

find_package(LinphoneCxx CONFIG QUIET)
find_package(Linphone CONFIG QUIET)
find_package(bctoolbox CONFIG QUIET)
find_package(belcard CONFIG QUIET)
find_package(Mediastreamer2 CONFIG QUIET)
find_package(ortp CONFIG QUIET)


set(LIBRARIES_LIST ${BCTOOLBOX_CORE_LIBRARIES} ${BELCARD_LIBRARIES}  ${LINPHONE_LIBRARIES} ${LINPHONECXX_LIBRARIES} ${MEDIASTREAMER2_LIBRARIES} ${ORTP_LIBRARIES} ${OPUS_LIBRARIES})
if(WIN32)
	set(LIBRARIES)
	foreach(LIBRARY ${LIBRARIES_LIST})# Search for lib full path
		find_library(FIND_LIBRARY_ITEM_${LIBRARY} NAMES ${LIBRARY} lib${LIBRARY} REQUIRED)#find_library need a specific variable name each time
		if(FIND_LIBRARY_ITEM_${LIBRARY})
			list(APPEND LIBRARIES ${FIND_LIBRARY_ITEM_${LIBRARY}})
		else()
			message(SEND_ERROR "${LIBRARY} not found!")
		endif()
	endforeach()
else()
	set(LIBRARIES ${LIBRARIES_LIST})
endif()

include_directories(${LINPHONE_OUTPUT_DIR}/include)

add_executable(main main.cpp)
target_link_libraries(main ${LIBRARIES})
