# Each Makefile lines are reported in this file with "->" 
# and the CMake implementation follows

# -> CC := gcc 
# -> BIN := posix_demo
project(posix_demo C)
cmake_minimum_required(VERSION 3.5)

# -> BUILD_DIR := build
set(EXECUTABLE_OUTPUT_PATH ${PROJECT_BINARY_DIR}/build)

# For debug, no change detected ...
set(CMAKE_VERBOSE_MAKEFILE ON)

# -> FREERTOS_DIR_REL := ./FreeRTOS/FreeRTOS
# -> FREERTOS_DIR := $(abspath $(FREERTOS_DIR_REL))
set(FREERTOS_DIR "${CMAKE_CURRENT_SOURCE_DIR}/FreeRTOS/FreeRTOS")

# -> INCLUDE_DIRS := -I.
# -> INCLUDE_DIRS += -I${FREERTOS_DIR}/Source/include
# -> INCLUDE_DIRS += -I${FREERTOS_DIR}/Source/portable/ThirdParty/GCC/Posix
# -> INCLUDE_DIRS += -I${FREERTOS_DIR}/Source/portable/ThirdParty/GCC/Posix/utils
# -> INCLUDE_DIRS += -I${FREERTOS_DIR}/Demo/Common/include
# -> INCLUDE_DIRS += -I${FREERTOS_DIR}/Demo/Posix_GCC
include_directories(
    .
    ${FREERTOS_DIR}/Source/include
    ${FREERTOS_DIR}/Source/portable/ThirdParty/GCC/Posix
    ${FREERTOS_DIR}/Source/portable/ThirdParty/GCC/Posix/utils
    ${FREERTOS_DIR}/Demo/Common/include
    ${FREERTOS_DIR}/Demo/Posix_GCC
)

# -> SOURCE_FILES := $(wildcard *.c)
# -> SOURCE_FILES += $(wildcard ${FREERTOS_DIR}/Source/*.c)
FILE(GLOB FREERTOS_SOURCES
    ${FREERTOS_DIR}/Source/*.c
)

# Memory manager (use malloc() / free())
# -> SOURCE_FILES += ${FREERTOS_DIR}/Source/portable/MemMang/heap_3.c
list(APPEND FREERTOS_SOURCES "${FREERTOS_DIR}/Source/portable/MemMang/heap_3.c")

# Posix port
# -> SOURCE_FILES += ${FREERTOS_DIR}/Source/portable/ThirdParty/GCC/Posix/utils/wait_for_event.c
# -> SOURCE_FILES += ${FREERTOS_DIR}/Source/portable/ThirdParty/GCC/Posix/port.c
list(APPEND FREERTOS_SOURCES "${FREERTOS_DIR}/Source/portable/ThirdParty/GCC/Posix/utils/wait_for_event.c")
list(APPEND FREERTOS_SOURCES "${FREERTOS_DIR}/Source/portable/ThirdParty/GCC/Posix/port.c")

# -> CFLAGS := -ggdb3 -O0 -DprojCOVERAGE_TEST=0 -D_WINDOWS_
add_definitions(-DprojCOVERAGE_TEST=0)
add_definitions(-D_WINDOWS_)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -ggdb3 -O0")

# -> LDFLAGS := -ggdb3 -O0 -pthread
# pthread included after
set(CMAKE_SHARED_LINKER_FLAGS "${CMAKE_SHARED_LINKER_FLAGS} -ggdb3 -O0")

# Debug
message("Project: ${PROJECT_NAME}")
message("Sources: ${FREERTOS_SOURCES}")

# -> End of Makefile : Compilation

# Create executable with demo code files
#add_executable(${PROJECT_NAME}
#    main.c
#    FreeRTOSConfig.h
#    console.h console.c
#)
# 遍历项目根目录下所有的 .cpp 文件
file (GLOB files tutorials/*.c)
foreach (file ${files})
    string(REGEX REPLACE ".+/(.+)\\..*" "\\1" exe ${file})
    add_executable (${exe} ${file}
            FreeRTOSConfig.h
            console.h console.c
            )
    message (\ \ \ \ --\ src/${exe}.cpp\ will\ be\ compiled\ to\ bin/${exe})
endforeach ()

# Link FreeRTOS as a static library
add_library(freertos STATIC ${FREERTOS_SOURCES})
#target_link_libraries(${PROJECT_NAME} freertos)
target_link_libraries(${exe} freertos)

## Add pthread
#target_link_libraries(${PROJECT_NAME} pthread)
target_link_libraries(${exe} pthread)
