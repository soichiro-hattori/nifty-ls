cmake_minimum_required(VERSION 3.15...3.27)
project(${SKBUILD_PROJECT_NAME} LANGUAGES CXX)

# put a symlink to the compile_commands.json in the build dir
# so the LSP can find it in a known directory without the platform tag
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)
execute_process(
    COMMAND ${CMAKE_COMMAND} -E create_symlink
        ${CMAKE_CURRENT_BINARY_DIR}/compile_commands.json
        ${CMAKE_CURRENT_BINARY_DIR}/../compile_commands.json
)

find_package(Python 3.8 COMPONENTS Interpreter Development.Module REQUIRED)
find_package(nanobind CONFIG REQUIRED)

option(NIFTY_LS_OPENMP "Enable OpenMP support in the compiled extensions" ON)

if(NIFTY_LS_OPENMP)
    find_package(OpenMP REQUIRED)
endif()

nanobind_add_module(cpu_helpers src/nifty_ls/cpu_helpers.cpp NOMINSIZE)

target_compile_options(cpu_helpers PRIVATE
    -Wall -Wextra -Werror -std=c++17
)

if (NOT CMAKE_BUILD_TYPE MATCHES Debug)
    target_compile_options(cpu_helpers PRIVATE -Ofast)
endif()

if (OpenMP_CXX_FOUND)
    target_link_libraries(cpu_helpers PRIVATE OpenMP::OpenMP_CXX)
endif()

install(TARGETS cpu_helpers LIBRARY DESTINATION nifty_ls)
