cmake_minimum_required(VERSION 3.13.4)
project(master_arbeit)

set(CMAKE_CXX_STANDARD 20)

if( NOT CMAKE_BUILD_TYPE )
    set( CMAKE_BUILD_TYPE Release)
endif()

set(CMAKE_MODULE_PATH
        ${CMAKE_CURRENT_SOURCE_DIR}/cmake/
        ${CMAKE_CURRENT_SOURCE_DIR}/vendor/liboscar/cmake/
        ${CMAKE_CURRENT_SOURCE_DIR}/vendor/liboscar/vendor/sserialize/cmake/
        ${CMAKE_MODULE_PATH}
)

option(PATHFINDER_OPTIMIZE_ULTRA "Build oscar-web with lto and aggressive optimization settings for sserialize")
if (PATHFINDER_OPTIMIZE_ULTRA)
	set(LIBOSCAR_NO_DATA_REFCOUNTING_ENABLED TRUE)
	set(SSERIALIZE_INLINE_IN_LTO_ENABLED TRUE)
	set(SSERIALIZE_CONTIGUOUS_UBA_ONLY_SOFT_FAIL_ENABLED TRUE)
	set(CMAKE_INTERPROCEDURAL_OPTIMIZATION TRUE)
endif()

option(PATHFINDER_BUILD_WITH_SANITIZERS "Build binaries with additional instrumentation (address, undefined behaviour, thread) sanitizer" OFF)
if (PATHFINDER_BUILD_WITH_SANITIZERS)
	set(MY_C_FLAGS "${MY_C_FLAGS} -g -fsanitize=address -fsanitize=undefined -fno-omit-frame-pointer")
	set(MY_CXX_FLAGS "${MY_C_FLAGS} -g -fsanitize=address -fsanitize=undefined -fno-omit-frame-pointer")

	set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} ${MY_C_FLAGS}")
	set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${MY_CXX_FLAGS}")
	set(CMAKE_C_FLAGS_DEBUG "${CMAKE_C_FLAGS_DEBUG} ${DEBUG_FLAGS}")
	set(CMAKE_CXX_FLAGS_DEBUG "${CMAKE_CXX_FLAGS_DEBUG} ${DEBUG_FLAGS}")
	set(CMAKE_C_FLAGS_RELWITHDEBINFO "${CMAKE_C_FLAGS_RELWITHDEBINFO} ${DEBUG_FLAGS}")
	set(CMAKE_CXX_FLAGS_RELWITHDEBINFO "${CMAKE_CXX_FLAGS_RELWITHDEBINFO} ${DEBUG_FLAGS}")
endif()



set(THREADS_PREFER_PTHREAD_FLAG ON)
find_package(Threads REQUIRED)

add_subdirectory(benchmarks)
add_subdirectory(vendor/path_finder)
add_subdirectory(vendor/oscar-routing)

add_executable(file_creator FileCreator.cpp CLI11.hpp)
add_executable(run_from_files RunFromFiles.cpp)

target_link_libraries(file_creator PRIVATE path_finder oscar-routing Threads::Threads)
target_link_libraries(run_from_files PRIVATE path_finder oscar-routing Threads::Threads)
