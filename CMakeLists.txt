
cmake_minimum_required(VERSION 3.16)

project(libepir-sodium-wasm C)

set(CMAKE_C_STANDARD 11)
set(CMAKE_POSITION_INDEPENDENT_CODE ON)
add_compile_options(-Wall -Wextra -Werror)

include(ExternalProject)

# Build libsodium.
set(LIBSODIUM_GIT_REPOSITORY "https://github.com/EllipticPIR/libsodium.git")
set(LIBSODIUM_GIT_TAG ci-master)
set(LIBSODIUM_CONFIGURE_CACHE libsodium_js.cache)
ExternalProject_Add(libsodium
	GIT_REPOSITORY ${LIBSODIUM_GIT_REPOSITORY}
	GIT_TAG ${LIBSODIUM_GIT_TAG}
	BUILD_IN_SOURCE on
	CONFIGURE_COMMAND
		emconfigure ./configure
			--cache-file=${LIBSODIUM_CONFIGURE_CACHE} --disable-shared --prefix=${CMAKE_CURRENT_BINARY_DIR}/libsodium
			--without-pthreads --disable-ssp --disable-asm --disable-pie "CFLAGS=-flto=full -O3"
	BUILD_COMMAND emmake $(MAKE)
	INSTALL_COMMAND emmake $(MAKE) install
)

# libsodium.js
add_custom_command(
	OUTPUT ${CMAKE_CURRENT_BINARY_DIR}/libsodium/lib/libsodium.js
	#COMMAND emcc -s MODULARIZE -s ALLOW_MEMORY_GROWTH=1 -s ALLOW_TABLE_GROWTH=1 -s SINGLE_FILE=1 -s EXPORTED_FUNCTIONS=['_malloc, _free'] -s EXPORTED_RUNTIME_METHODS=['addFunction, removeFunction'] -s EXPORT_ALL=1 -s LINKABLE=1 -flto=full -O3 ./libsodium/lib/libsodium.a -o ${CMAKE_CURRENT_BINARY_DIR}/libsodium/lib/libsodium.js
	COMMAND emcc
	-s MODULARIZE
	-s ALLOW_MEMORY_GROWTH=1
	-s ALLOW_TABLE_GROWTH=1
	-s SINGLE_FILE=1
	-s EXPORTED_FUNCTIONS=['_malloc, _free']
	-s EXPORTED_RUNTIME_METHODS=['addFunction, removeFunction']
	-s EXPORT_ALL=1
	-s LINKABLE=1
	-flto=full -O3
	./libsodium/lib/libsodium.a -o ${CMAKE_CURRENT_BINARY_DIR}/libsodium/lib/libsodium.js
	DEPENDS libsodium
)
add_custom_target(libsodium_js ALL DEPENDS ${CMAKE_CURRENT_BINARY_DIR}/libsodium/lib/libsodium.js)
install(
	FILES ${CMAKE_CURRENT_BINARY_DIR}/libsodium/lib/libsodium.a
	      ${CMAKE_CURRENT_BINARY_DIR}/libsodium/lib/libsodium.la
	      ${CMAKE_CURRENT_BINARY_DIR}/libsodium/lib/libsodium.js
	DESTINATION .
)

