cmake_minimum_required(VERSION 3.13...3.17)
include(ExternalProject)
#include(FetchContent)

project(UnoSemuxLightCoreWasm
	VERSION 0.1.0
	DESCRIPTION "C++ common utilities and types library"
	LANGUAGES CXX)

set(CMAKE_VERBOSE_MAKEFILE		ON)
set(CMAKE_BUILD_PARALLEL_LEVEL	3)

add_executable(${PROJECT_NAME}
	Wallet/UnoSemuxAccountHDWasm.cpp
	Wallet/UnoSemuxAddrWasm.cpp
	Wallet/UnoSemuxNetworkTypeWasm.cpp
	Wallet/UnoSemuxTransactionSignWasm.cpp
	Wallet/UnoSemuxTransactionTypeWasm.cpp
	Wallet/UnoSemuxTransactionWasm.cpp
	WasmUtils/UnoSemuxWasmUtils.cpp
	main.cpp
	)

add_compile_definitions(
	UNOSEMUXLIGHTCOREWASM_LIBRARY
	HAVE_NETINET_IN_H
	_GLIBCXX_USE_NANOSLEEP
	)

set(TOOLS						"/cpp/emsdk/upstream/emscripten")
set(CMAKE_TOOLCHAIN_FILE		"${TOOLS}/cmake/Modules/Platform/Emscripten.cmake")
set(CMAKE_CXX_COMPILER			${TOOLS}/em++)
set(CMAKE_C_COMPILER			${TOOLS}/emcc)
set(CMAKE_CXX_STANDARD			20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

target_compile_options(${PROJECT_NAME} PRIVATE -std=gnu++2a)
target_compile_options(${PROJECT_NAME} PRIVATE -fexceptions)
target_compile_options(${PROJECT_NAME} PRIVATE -s WASM=1 -s MAIN_MODULE=1)
target_compile_options(${PROJECT_NAME} PRIVATE -O3)

target_compile_options(${PROJECT_NAME} PRIVATE -fvisibility=hidden -fvisibility-inlines-hidden)
target_compile_options(${PROJECT_NAME} PRIVATE -ffunction-sections -fdata-sections)
#target_compile_options(${PROJECT_NAME} PRIVATE -fno-rtti)
target_link_options(${PROJECT_NAME} PRIVATE -Wl)
set_target_properties(${PROJECT_NAME} PROPERTIES LINK_FLAGS "-s DEMANGLE_SUPPORT=1 --preload-file assets --bind")

target_compile_definitions(${PROJECT_NAME} PRIVATE __EMSCRIPTEN__)

message("-- [${PROJECT_NAME}]: ***************** Build mode RELEASE *****************")
set(OUT_BUILD_MODE_PATH Release)
target_compile_definitions(${PROJECT_NAME} PRIVATE RELEASE_BUILD)

#------------------------ ARC ---------------------
set(CMAKE_HOST_SYSTEM_PROCESSOR wasm32)

set(PROJECT_BINARY_DIR		"${CMAKE_SOURCE_DIR}/../../bin")
set(PROJECT_LIBRARY_DIR		"${CMAKE_SOURCE_DIR}/../../lib")
set(EXTRAS					"${CMAKE_SOURCE_DIR}/../../include")
set(PREFIX_FOR_EXTERNAL_TMP "${CMAKE_SOURCE_DIR}/../../tmp")

file(MAKE_DIRECTORY ${EXTRAS})

set(SP _)
set(BUILD_NAME_DIR	"${CMAKE_BUILD_TYPE}${SP}${CMAKE_SYSTEM_NAME}${SP}${CMAKE_HOST_SYSTEM_PROCESSOR}")
set(DESTDIR			"${PROJECT_BINARY_DIR}/${BUILD_NAME_DIR}")
set(LIBDIR			"${PROJECT_LIBRARY_DIR}/${CMAKE_BUILD_TYPE}${SP}Linux${SP}x86_64")

set(CMAKE_BINARY_DIR				${DESTDIR})
set(EXECUTABLE_OUTPUT_PATH			${CMAKE_BINARY_DIR})
set(LIBRARY_OUTPUT_PATH				${CMAKE_BINARY_DIR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY	${CMAKE_BINARY_DIR})

set(GPCORE2			GpCore2)
set(UTF8PROC		utf8proc)
set(GPCRYPTOCORE	GpCryptoCore)
set(LIBSODIUM		sodium)

set(SEMUXLIGHTCORE  UnoSemuxLightCore)
if (NOT EXISTS ${DESTDIR}/lib${SEMUXLIGHTCORE}.so)
	message("-- [${PROJECT_NAME}]: ${SEMUXLIGHTCORE} not found")
	ExternalProject_Add(${SEMUXLIGHTCORE}_ext
		PREFIX			  "."
		DOWNLOAD_DIR	  "${CMAKE_SOURCE_DIR}/../${SEMUXLIGHTCORE}"
		SOURCE_DIR		  "${CMAKE_SOURCE_DIR}/../${SEMUXLIGHTCORE}"
		STAMP_DIR		  "${PREFIX_FOR_EXTERNAL_TMP}"
		TMP_DIR			  "${PREFIX_FOR_EXTERNAL_TMP}"
		GIT_REPOSITORY    https://github.com/bvchirkov/UnoSemuxLightCore.git
#		GIT_TAG			  aa49a351d06927fcc9e4e5d15ab73a78bdbe3f5f
		CMAKE_ARGS		  -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} -DCMAKE_CXX_COMPILER=g++-9
		BUILD_IN_SOURCE   ON
		INSTALL_COMMAND	  make -j4 install
	)
endif()

target_include_directories(${PROJECT_NAME}
	PRIVATE ${EXTRAS}
	PRIVATE ${EXTRAS}/${SEMUXLIGHTCORE}
	PRIVATE "/cpp/emsdk/upstream/emscripten/system/include"
	)

target_link_directories(${PROJECT_NAME}
	PRIVATE ${LIBDIR}
	)

target_link_libraries(${PROJECT_NAME}
	PRIVATE
		${LIBSODIUM}
		${GPCORE2}
		${UTF8PROC}
		${GPCRYPTOCORE}
		${SEMUXLIGHTCORE}
	)

message("-- [${PROJECT_NAME}]: Target name:              " ${PROJECT_NAME}${TARGET})
message("-- [${PROJECT_NAME}]: Target arch:              " ${CMAKE_HOST_SYSTEM_PROCESSOR})
message("-- [${PROJECT_NAME}]: Target OS:                " ${CMAKE_SYSTEM_NAME})
message("-- [${PROJECT_NAME}]: Build directory:          " ${DESTDIR})
message("-- [${PROJECT_NAME}]: C++ standart:             " ${CMAKE_CXX_STANDARD})
message("-- [${PROJECT_NAME}]: GCC C compiler version:   " ${CMAKE_C_COMPILER})
message("-- [${PROJECT_NAME}]: GCC C++ compiler version: " ${CMAKE_CXX_COMPILER})

#set_property(TARGET ${PROJECT_NAME} PROPERTY VERSION ${PROJECT_VERSION})
#set_property(TARGET ${PROJECT_NAME} PROPERTY SOVERSION ${PROJECT_VERSION_MAJOR})
