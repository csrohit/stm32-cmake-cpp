cmake_minimum_required(VERSION 3.24)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

set(CMAKE_VERBOSE_MAKEFILE ON)

set(CMAKE_SYSTEM_NAME Generic)
set(CMAKE_SYSTEM_PROCESSOR arm)

set(TOOLCHAIN_PREFIX arm-none-eabi-)
set(FLAGS
    "-nostdlib -fno-tree-loop-distribute-patterns")

set(CMAKE_C_COMPILER ${TOOLCHAIN_PREFIX}gcc ${FLAGS})
set(CMAKE_ASM_COMPILER ${CMAKE_C_COMPILER})
set(CMAKE_CXX_COMPILER ${TOOLCHAIN_PREFIX}g++ ${FLAGS} ${CPP_FLAGS})
set(CMAKE_OBJCOPY ${TOOLCHAIN_PREFIX}objcopy)
set(CMAKE_SIZE ${TOOLCHAIN_PREFIX}size)
set(CMAKE_OBJDUMP ${TOOLCHAIN_PREFIX}objdump)

set(CMAKE_EXECUTABLE_SUFFIX_ASM ".elf")
set(CMAKE_EXECUTABLE_SUFFIX_C ".elf")
set(CMAKE_EXECUTABLE_SUFFIX_CXX ".elf")


set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)



# set project language
project(systick LANGUAGES CXX C)
set(EXECUTABLE ${CMAKE_PROJECT_NAME})

set(MCU_MODEL STM32F103xB)
set(CPU_PARAMETERS
    -mfloat-abi=soft
    -mthumb
    -mcpu=cortex-m3
)

set(MCU_LINKER_SCRIPT ${CMAKE_SOURCE_DIR}/stm32f1.ld)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_STANDARD 11)
set(CMAKE_C_EXTENSIONS OFF)

set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_EXTENSIONS ON)

add_executable(${EXECUTABLE}
    src/main.cpp
    src/startup_stm32f103.cpp
    src/gpio.cpp
)

target_compile_definitions(${EXECUTABLE}
    PRIVATE
    STM32F103xB
)


# add include directories 
target_include_directories(${EXECUTABLE} PRIVATE
    ${CMAKE_SOURCE_DIR}/include    
)


# add flags for compiler
target_compile_options(${EXECUTABLE}
    PRIVATE
        ${CPU_PARAMETERS}
        -Wall
        -Wextra
        -Wundef
        -Wshadow
        -Wredundant-decls
        -fdata-sections 
        -ffunction-sections 
        -fno-rtti 
        -fno-exceptions
)

# add flags for linker
target_link_options(${EXECUTABLE} 
    PRIVATE
        -T ${MCU_LINKER_SCRIPT}
        ${CPU_PARAMETERS}
        -Wl,-Map=${CMAKE_PROJECT_NAME}.map
)

# add target for flashing the micro-controller 
add_custom_target(flash
    COMMAND st-flash write ${EXECUTABLE}.bin 0x8000000
)

add_custom_command(TARGET ${EXECUTABLE} POST_BUILD
    COMMAND ${CMAKE_SIZE} $<TARGET_FILE:${EXECUTABLE}>
    COMMAND ${CMAKE_OBJCOPY} -O ihex $<TARGET_FILE:${EXECUTABLE}> ${EXECUTABLE}.hex
    COMMAND ${CMAKE_OBJCOPY} -O binary $<TARGET_FILE:${EXECUTABLE}> ${EXECUTABLE}.bin
)

set(CMAKE_TRY_COMPILE_TARGET_TYPE STATIC_LIBRARY)
