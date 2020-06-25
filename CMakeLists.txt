cmake_minimum_required(VERSION 3.10)
project(hashensemble)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall -O3")
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -O3")

message("${CMAKE_CXX_FLAGS}")

include_directories(src 3rd/include)

find_package(Boost
        REQUIRED
        COMPONENTS system serialization
        )

include_directories(${Boost_INCLUDE_DIRS})

find_package(fmt REQUIRED)
include_directories(${fmt_INCLUDE_DIRS})

file(GLOB external_csrc "./3rd/include/*.c")
file(GLOB external_cxxsrc "./3rd/include/*.cpp")

set(external_objs "")

foreach(csrc ${external_csrc})
    message("Compiling ${csrc} ...")
    get_filename_component(objname ${csrc} NAME_WE)
    add_library(${objname} OBJECT ${csrc})
    list(APPEND external_objs $<TARGET_OBJECTS:${objname}>)
endforeach()

foreach(cxxsrc ${external_cxxsrc})
    message("Compiling ${cxxsrc} ...")
    get_filename_component(objname ${cxxsrc} NAME_WE)
    add_library(${objname} OBJECT ${cxxsrc})
    list(APPEND external_objs $<TARGET_OBJECTS:${objname}>)
endforeach()

message("external_objs = ${external_objs} ...")


enable_testing()
add_executable(example test/hashensemble_example.cpp ${external_objs})
add_test(NAME example
         COMMAND $<TARGET_FILE:example>)
add_test(NAME vexample
        COMMAND valgrind --leak-check=yes $<TARGET_FILE:example>)

add_executable(benchmark_rt benchmarks/hashensemble_benchmark.cpp ${external_objs})
target_include_directories(benchmark_rt PUBLIC benchmarks)
target_link_libraries(benchmark_rt Boost::system Boost::serialization fmt::fmt)

add_executable(benchmark_ct_xxh benchmarks/hashensemble_benchmark.cpp ${external_objs})
target_compile_definitions(benchmark_ct_xxh PUBLIC -DUSE_XXH)
target_include_directories(benchmark_ct_xxh PUBLIC benchmarks)
target_link_libraries(benchmark_ct_xxh Boost::system Boost::serialization fmt::fmt)

add_executable(benchmark_ct_wyh benchmarks/hashensemble_benchmark.cpp ${external_objs})
target_compile_definitions(benchmark_ct_wyh PUBLIC -DUSE_WYH -DVERBOSE_LEVEL=2)
target_include_directories(benchmark_ct_wyh PUBLIC benchmarks)
target_link_libraries(benchmark_ct_wyh Boost::system Boost::serialization fmt::fmt)

add_custom_target(Name benchmark
        COMMAND $<TARGET_FILE:benchmark_rt>
        COMMAND $<TARGET_FILE:benchmark_ct_xxh>
        COMMAND $<TARGET_FILE:benchmark_ct_wyh>
        DEPENDS benchmark_rt benchmark_ct_xxh benchmark_ct_wyh
        WORKING_DIRECTORY ${CMAKE_CURRENT_BINARY_DIR})