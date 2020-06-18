cmake_minimum_required(VERSION 3.10...3.17)
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

target_compile_definitions(${PROJECT_NAME} PRIVATE
	UNOSEMUXLIGHTCOREWASM_LIBRARY
	HAVE_NETINET_IN_H
	_GLIBCXX_USE_NANOSLEEP
	__EMSCRIPTEN__
	RELEASE_BUILD
	NDEBUG
	)

#set(EMSCRIPTEN_ROOT_PATH		"/cpp/emsdk/upstream/emscripten")
set(CMAKE_TOOLCHAIN_FILE		"${EMSCRIPTEN_ROOT_PATH}/cmake/Modules/Platform/Emscripten.cmake")
set(CMAKE_CXX_COMPILER			${EMSCRIPTEN_ROOT_PATH}/em++)
set(CMAKE_C_COMPILER			${EMSCRIPTEN_ROOT_PATH}/emcc)
set(CMAKE_CXX_STANDARD			20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(CMAKE_C_FLAGS_RELEASE				"" CACHE STRING "Uno-labs-overridden CMAKE_C_FLAGS_RELEASE" FORCE)
set(CMAKE_SHARED_LINKER_FLAGS_RELEASE	"" CACHE STRING "Uno-labs-overridden CMAKE_SHARED_LINKER_FLAGS_RELEASE" FORCE)
set(CMAKE_MODULE_LINKER_FLAGS_RELEASE	"" CACHE STRING "Uno-labs-overridden CMAKE_SHARED_LINKER_FLAGS_RELEASE" FORCE)
set(CMAKE_EXE_LINKER_FLAGS_RELEASE		"" CACHE STRING "Uno-labs-overridden CMAKE_SHARED_LINKER_FLAGS_RELEASE" FORCE)
set(CMAKE_CXX_FLAGS_RELEASE				"-std=c++2a -O3"
	CACHE STRING "Uno-labs-overridden CMAKE_CXX_FLAGS_RELEASE" FORCE)

string(JOIN " " COMPILE_FLAGS
	-s WASM=1
	-s EXTRA_EXPORTED_RUNTIME_METHODS=['ALLOC_NORMAL','FS','UTF8ToString','allocate','ccall','getValue','setValue']
	-s ALLOW_MEMORY_GROWTH=1

	-fexceptions
	-fvisibility=hidden
	-fvisibility-inlines-hidden
	-ffunction-sections
	-fdata-sections
	)

string(JOIN " " LINK_FLAGS
	-s WASM=1

	-Wl

	--gc-sections
	--bind
	)

set_target_properties(${PROJECT_NAME} PROPERTIES COMPILE_FLAGS ${COMPILE_FLAGS})
set_target_properties(${PROJECT_NAME} PROPERTIES LINK_FLAGS ${LINK_FLAGS})

set(PROJECT_BINARY_DIR		"${CMAKE_SOURCE_DIR}/../../bin")
set(PROJECT_LIBRARY_DIR		"${CMAKE_SOURCE_DIR}/../../lib")
set(EXTRAS					"${CMAKE_SOURCE_DIR}/../../include")
set(PREFIX_FOR_EXTERNAL_TMP "${CMAKE_SOURCE_DIR}/../../tmp")

set(DESTDIR			"${PROJECT_BINARY_DIR}")
set(LIBDIR			"${PROJECT_LIBRARY_DIR}")

set(CMAKE_BINARY_DIR				${DESTDIR})
set(EXECUTABLE_OUTPUT_PATH			${CMAKE_BINARY_DIR})
set(LIBRARY_OUTPUT_PATH				${CMAKE_BINARY_DIR})
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY	${CMAKE_BINARY_DIR})

set(GPCORE2			GpCore2)
set(UTF8PROC		utf8proc)
set(GPCRYPTOCORE	GpCryptoCore)
set(LIBSODIUM		sodium)

set(SEMUXLIGHTCORE  UnoSemuxLightCore)
if (NOT EXISTS "${CMAKE_SOURCE_DIR}/../${SEMUXLIGHTCORE}")
	message(STATUS "[${PROJECT_NAME}]: ${SEMUXLIGHTCORE} not found")
	ExternalProject_Add(${SEMUXLIGHTCORE}_ext
		PREFIX			  "."
		DOWNLOAD_DIR	  "${CMAKE_SOURCE_DIR}/../${SEMUXLIGHTCORE}"
		SOURCE_DIR		  "${CMAKE_SOURCE_DIR}/../${SEMUXLIGHTCORE}"
		STAMP_DIR		  "${PREFIX_FOR_EXTERNAL_TMP}"
		TMP_DIR			  "${PREFIX_FOR_EXTERNAL_TMP}"
		GIT_REPOSITORY    https://github.com/bvchirkov/UnoSemuxLightCore.git
#		GIT_TAG			  aa49a351d06927fcc9e4e5d15ab73a78bdbe3f5f
		CMAKE_ARGS		  -DCMAKE_BUILD_TYPE=${CMAKE_BUILD_TYPE} -DEMSCRIPTEN_ROOT_PATH=${EMSCRIPTEN_ROOT_PATH}
		BUILD_IN_SOURCE   ON
		INSTALL_COMMAND	  ${EMSCRIPTEN_ROOT_PATH}/emmake make -j4 install
	)
endif()

target_include_directories(${PROJECT_NAME}
	PRIVATE ${EXTRAS}
	PRIVATE ${EXTRAS}/${SEMUXLIGHTCORE}
	)

target_link_directories(${PROJECT_NAME}
	PUBLIC ${LIBDIR}
	)

target_link_libraries(${PROJECT_NAME}
	PUBLIC
		${LIBSODIUM}
		${GPCORE2}
		${UTF8PROC}
		${GPCRYPTOCORE}
		${SEMUXLIGHTCORE}
	)

message(STATUS "[${PROJECT_NAME}]: Target name:              " ${PROJECT_NAME}${TARGET})
message(STATUS "[${PROJECT_NAME}]: Target arch:              " ${CMAKE_SYSTEM_PROCESSOR})
message(STATUS "[${PROJECT_NAME}]: Target OS:                " ${CMAKE_SYSTEM_NAME})
message(STATUS "[${PROJECT_NAME}]: Build directory:          " ${DESTDIR})
message(STATUS "[${PROJECT_NAME}]: C++ standart:             " ${CMAKE_CXX_STANDARD})
message(STATUS "[${PROJECT_NAME}]: GCC C compiler version:   " ${CMAKE_C_COMPILER})
message(STATUS "[${PROJECT_NAME}]: GCC C++ compiler version: " ${CMAKE_CXX_COMPILER})
