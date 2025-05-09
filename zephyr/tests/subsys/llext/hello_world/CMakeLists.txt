# Copyright (c) 2023 Intel Corporation.
# SPDX-License-Identifier: Apache-2.0

cmake_minimum_required(VERSION 3.20.0)
find_package(Zephyr REQUIRED HINTS $ENV{ZEPHYR_BASE})
project(hello_world)

if(NOT CONFIG_MODULES OR CONFIG_LLEXT_TEST_HELLO STREQUAL "m")

# TODO check which architecture is being used
if(CONFIG_ARM)
    set(CMAKE_C_FLAGS "-mlong-calls" "-mthumb")

    add_custom_command(OUTPUT ${PROJECT_BINARY_DIR}/hello_world.llext
	COMMAND ${CMAKE_C_COMPILER} ${CMAKE_C_FLAGS} -c
		-I ${PROJECT_SOURCE_DIR}/../../../../include
		-imacros${PROJECT_BINARY_DIR}/../zephyr/include/generated/autoconf.h
		-o ${PROJECT_BINARY_DIR}/hello_world.llext
		${PROJECT_SOURCE_DIR}/hello_world.c
    )
elseif(CONFIG_XTENSA)
    set(CMAKE_C_FLAGS "-shared" "-fPIC" "-nostdlib" "-nodefaultlibs")

    add_custom_command(OUTPUT ${PROJECT_BINARY_DIR}/hello_world.llext
	COMMAND ${CMAKE_C_COMPILER} ${CMAKE_C_FLAGS}
		-I ${PROJECT_SOURCE_DIR}/../../../../include
		-imacros${PROJECT_BINARY_DIR}/../zephyr/include/generated/autoconf.h
		-o ${PROJECT_BINARY_DIR}/hello_world.pre.llext
		${PROJECT_SOURCE_DIR}/hello_world.c
	COMMAND ${CROSS_COMPILE}strip -R .xt.*
		-o ${PROJECT_BINARY_DIR}/hello_world.llext
		${PROJECT_BINARY_DIR}/hello_world.pre.llext
    )
endif()

set(HELLO_WORLD_LLEXT ${PROJECT_BINARY_DIR}/hello_world.llext PARENT_SCOPE)

add_custom_target(hello_world DEPENDS ${PROJECT_BINARY_DIR}/hello_world.llext)

endif()
