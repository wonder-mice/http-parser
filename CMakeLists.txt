cmake_minimum_required(VERSION 3.2)

if(NOT DEFINED CMAKE_BUILD_TYPE)
	set(CMAKE_BUILD_TYPE Release CACHE STRING "Build type")
endif()

# The most common usecase for CMakeLists.txt in a library project is to be
# included with add_subdirectory() into another project. However there also
# must be a way to build a library alone. Common way is to separate those
# two cases by providing two CMakeLists.txt files where one includes another
# with add_subdirectory(). Corresponding directory structure may look like:
#   http-parser/
#       http-parser/  (src is also a common name)
#           http_parser.c
#           CMakeLists.txt (can be used by other project with add_subdirectory)
#       CMakeLists.txt (standalone, also uses add_subdirectory(http-parser))
# That avoids polution of master project with unnecessary targets and cache
# variables. Current directory structure can't be used to apply the same
# technique. Instead, this weird variable will be used. Projects that want to
# use this CMakeLists.txt file with add_subdirectory() will need to
#   set(HTTP_PARSER_STANDALONE OFF)
# before calling  add_subdirectory().
option(HTTP_PARSER_STANDALONE "Standalone build" ON)
option(HTTP_PARSER_BUILD_SHARED "Build shared library" OFF)
option(HTTP_PARSER_BUILD_PARSERTRACE "Build parsertrace tool" OFF)
option(HTTP_PARSER_BUILD_URL_PARSER "Build url_parser tool" OFF)
option(HTTP_PARSER_BUILD_BENCH "Build bench tool" OFF)
option(HTTP_PARSER_BUILD_TESTS "Build tests" OFF)

set(BUILD_SHARED_LIBS ${HTTP_PARSER_BUILD_SHARED})

# Test that it matters
if(HTTP_PARSER_STANDALONE)
	project(http-parser)
endif()

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wall -Wextra -Werror")
if(CMAKE_BUILD_TYPE STREQUAL "Debug")
	set(HTTP_PARSER_STRICT 1)
else()
	set(HTTP_PARSER_STRICT 0)
endif()

add_library(http_parser http_parser.c http_parser.h)
set_target_properties(http_parser PROPERTIES VERSION "2.5.0")
target_compile_definitions(http_parser PRIVATE HTTP_PARSER_STRICT=${HTTP_PARSER_STRICT})

if(HTTP_PARSER_BUILD_TESTS)
	add_executable(tester test.c)
	target_link_libraries(tester http_parser)
	target_compile_definitions(tester PRIVATE HTTP_PARSER_STRICT=${HTTP_PARSER_STRICT})
	set_target_properties(tester PROPERTIES COMPILE_FLAGS -Wno-unused-parameter)

	enable_testing()
	add_test(NAME tester COMMAND tester)
endif()

if(HTTP_PARSER_BUILD_BENCH)
	add_executable(bench bench.c)
	target_link_libraries(bench http_parser)
	set_target_properties(bench PROPERTIES COMPILE_FLAGS -Wno-unused-parameter)
endif()

if(HTTP_PARSER_BUILD_PARSERTRACE)
	add_executable(parsertrace contrib/parsertrace.c)
	target_link_libraries(parsertrace http_parser)
	target_include_directories(parsertrace PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}")
endif()

if(HTTP_PARSER_BUILD_URL_PARSER)
	add_executable(url_parser contrib/url_parser.c)
	target_link_libraries(url_parser http_parser)
	target_include_directories(url_parser PRIVATE "${CMAKE_CURRENT_SOURCE_DIR}")
endif()

if(HTTP_PARSER_STANDALONE)
	add_custom_target(tags
		COMMAND ctags http_parser.c http_parser.h test.c
		WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}")
endif()
