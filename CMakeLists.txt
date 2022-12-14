# CMake template file
# Use this file with
# . ~/.vcpkg/vcpkg-init.ps1
# vcpkg activate
# cmake --preset m33-debug
# cmake --build --preset m33-debug

cmake_minimum_required(VERSION 3.20)

# Optional: print out extra messages to see what is going on. Comment it to have less verbose messages
set(CMAKE_VERBOSE_MAKEFILE ON)

# Setup project, output and linker file
project(LPC55S69_cmake_template)
set(EXECUTABLE ${PROJECT_NAME}.elf)
set(LINKER_FILE ${CMAKE_SOURCE_DIR}/device/LPC55S69.ld)

enable_language(C ASM)
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)
set(CMAKE_C_EXTENSIONS OFF)

# Optional: issue a message to be sure it uses the correct toolchain file.
message(STATUS "CMAKE_TOOLCHAIN_FILE is: ${CMAKE_TOOLCHAIN_FILE}")

# List of source files
set(SRC_FILES
        app/main.c
        startup/startup_lpc55s69_cm33_core0.c
        startup/boot_multicore_slave.c
        device/system_LPC55S69_cm33_core0.c
        )

# Build the executable based on the source files
add_executable(${EXECUTABLE} ${SRC_FILES})

# List of compiler defines, prefix with -D compiler option
target_compile_definitions(${EXECUTABLE} PRIVATE
        -DCPU_LPC55S69JBD100_cm33_core0 
        -DCPU_LPC55S69JBD100
        -DCPU_LPC55S69JBD100_cm33
        -D__MULTICORE_MASTER
        )

# List of includ directories
target_include_directories(${EXECUTABLE} PRIVATE
        . 
        device
        startup
        CMSIS
        )

# Compiler options
target_compile_options(${EXECUTABLE} PRIVATE
        -mcpu=cortex-m33
        -mthumb
        -mfpu=fpv5-sp-d16
        -mfloat-abi=hard

        -fdata-sections
        -ffunction-sections

        -Wall
        -O0
        -g3
        )

# Linker options
target_link_options(${EXECUTABLE} PRIVATE
        -T${LINKER_FILE}
        -mcpu=cortex-m33
        -mthumb
        -mfpu=fpv5-sp-d16
        -mfloat-abi=hard
        -specs=nano.specs
        -specs=nosys.specs
        -lc
        -lm
        -Wl,-Map=${PROJECT_NAME}.map,--cref
        -Wl,--gc-sections
        -Xlinker -print-memory-usage -Xlinker
        )

# Optional: Print executable size as part of the post build process
add_custom_command(TARGET ${EXECUTABLE}
        POST_BUILD
        COMMAND ${CMAKE_SIZE_UTIL} ${EXECUTABLE})

# Optional: Create hex, bin and S-Record files after the build
add_custom_command(TARGET ${EXECUTABLE}
        POST_BUILD
        COMMAND ${CMAKE_OBJCOPY} -O srec --srec-len=64 "${EXECUTABLE}" "$${PROJECT_NAME}.s19"
        COMMAND ${CMAKE_OBJCOPY} -O ihex "${EXECUTABLE}" "$${PROJECT_NAME}.hex"
        COMMAND ${CMAKE_OBJCOPY} -O binary "${EXECUTABLE}" "${PROJECT_NAME}.bin" 
        )
