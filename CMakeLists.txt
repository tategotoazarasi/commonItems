cmake_minimum_required(VERSION 3.25)
project(commonItems VERSION 1.0.0)

set(CMAKE_CXX_STANDARD 23)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

option(COVERAGE "Enable code coverage" OFF)

message("-- Configuring for platform: ${CMAKE_SYSTEM_NAME}")
message("-- Code coverage enabled: ${COVERAGE}")

if (CMAKE_SYSTEM_NAME STREQUAL "Windows")
	set(UNICODE_DEFAULT OFF)
	ADD_DEFINITIONS(-DUNICODE -D_UNICODE -DWINDOWS)
elseif (CMAKE_SYSTEM_NAME STREQUAL "Linux")
	if (COVERAGE)
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -O0 -fprofile-arcs -ftest-coverage -Wno-deprecated-declarations -Wno-unknown-pragmas")
		set(CMAKE_CXX_OUTPUT_EXTENSION_REPLACE ON)
	else()
		set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -g -O0 -Wall -Wno-deprecated-declarations -Wno-unknown-pragmas")
	endif()
endif()

find_package(nlohmann_json REQUIRED)
find_package(GTest REQUIRED)

include_directories(.)
if (CMAKE_SYSTEM_NAME STREQUAL "Linux")
	include_directories(/usr/include)
	link_directories(/usr/lib)
endif()

# Create OBJECT_DIR variable
set(OBJECT_DIR ${CMAKE_BINARY_DIR}/CMakeFiles/commonItemsTests.dir)
message("-- Object files will be output to: ${OBJECT_DIR}")

################################################################################
# Source groups
################################################################################
set(COMMON_SOURCES ${COMMON_SOURCES} "BulkParser.cpp")
set(COMMON_SOURCES ${COMMON_SOURCES} "Color.cpp")
set(COMMON_SOURCES ${COMMON_SOURCES} "CommonFunctions.cpp")
set(COMMON_SOURCES ${COMMON_SOURCES} "ConvenientParser.cpp")
set(COMMON_SOURCES ${COMMON_SOURCES} "ConverterVersion.cpp")
set(COMMON_SOURCES ${COMMON_SOURCES} "Date.cpp")
set(COMMON_SOURCES ${COMMON_SOURCES} "GameVersion.cpp")
set(COMMON_SOURCES ${COMMON_SOURCES} "iconvlite.cpp")
if (CMAKE_SYSTEM_NAME STREQUAL "Windows")
	set(COMMON_SOURCES ${COMMON_SOURCES} "WinUtils.cpp")
elseif (CMAKE_SYSTEM_NAME STREQUAL "Linux")
	set(COMMON_SOURCES ${COMMON_SOURCES} "LinuxUtils.cpp")
endif()
set(COMMON_SOURCES ${COMMON_SOURCES} "Log.cpp")
set(COMMON_SOURCES ${COMMON_SOURCES} "OSCommonLayer.cpp")
set(COMMON_SOURCES ${COMMON_SOURCES} "Parser.cpp")
set(COMMON_SOURCES ${COMMON_SOURCES} "ParserHelpers.cpp")
set(COMMON_SOURCES ${COMMON_SOURCES} "StringUtils.cpp")
set(COMMON_SOURCES ${COMMON_SOURCES} "Localization/LocalizationBlock.cpp")
set(COMMON_SOURCES ${COMMON_SOURCES} "Localization/LocalizationDatabase.cpp")
set(COMMON_SOURCES ${COMMON_SOURCES} "ModLoader/ModFilesystem.cpp")
set(COMMON_SOURCES ${COMMON_SOURCES} "ModLoader/ModLoader.cpp")
set(COMMON_SOURCES ${COMMON_SOURCES} "ModLoader/ModParser.cpp")
add_library(commonLib
            ${COMMON_SOURCES}
            )
add_library(commonItems::commonItems ALIAS commonLib)

target_include_directories(commonLib
                           PUBLIC
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/Localization>
                           $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/ModLoader>
                           $<INSTALL_INTERFACE:include>
                           )

add_subdirectory(external/zip SYSTEM PRIVATE EXCLUDE_FROM_ALL)
get_target_property(lib_include_dirs zip INTERFACE_INCLUDE_DIRECTORIES)
target_include_directories(commonLib SYSTEM PRIVATE ${lib_include_dirs})
target_link_libraries(commonLib PRIVATE zip)

target_link_libraries(commonLib PUBLIC nlohmann_json::nlohmann_json)

if (CMAKE_SYSTEM_NAME STREQUAL "Windows")
	target_compile_options(commonLib PRIVATE /permissive /W4 /w14242 /w14254 /w14263 /w14265 /w14287 /we4289 /w14296 /w14311 /w14545 /w14546
	                       /w14547 /w14549 /w14555 /w14619 /w14640 /w14826 /w14905 /w14906 /w14928 /external:anglebrackets /external:W0 /WX)
elseif (CMAKE_SYSTEM_NAME STREQUAL "Linux")
endif()

file(GLOB_RECURSE PUBLIC_HEADERS
     "${CMAKE_CURRENT_SOURCE_DIR}/*.h"
     "${CMAKE_CURRENT_SOURCE_DIR}/Localization/*.h"
     "${CMAKE_CURRENT_SOURCE_DIR}/ModLoader/*.h"
     )

include(GNUInstallDirs)

install(TARGETS commonLib zip
        EXPORT commonItemsTargets
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
        )
install(FILES ${PUBLIC_HEADERS}
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/commonItems
        )
install(EXPORT commonItemsTargets
        FILE commonItemsTargets.cmake
        NAMESPACE commonItems::
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/commonItems
        )
include(CMakePackageConfigHelpers)
write_basic_package_version_file(
		"${CMAKE_CURRENT_BINARY_DIR}/commonItemsConfigVersion.cmake"
		VERSION ${PROJECT_VERSION}
		COMPATIBILITY AnyNewerVersion
		)
install(FILES
        "${CMAKE_CURRENT_BINARY_DIR}/commonItemsConfigVersion.cmake"
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/commonItems
        )
configure_package_config_file(
		"cmake/commonItemsConfig.cmake.in"
		"${CMAKE_CURRENT_BINARY_DIR}/commonItemsConfig.cmake"
		INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/commonItems
		)
install(FILES "${CMAKE_CURRENT_BINARY_DIR}/commonItemsConfig.cmake"
        DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/commonItems
        )
export(PACKAGE commonItems)

set(tests
    "tests/test_main.cpp"
    "tests/ColorTests.cpp"
    "tests/CommonFunctionsTests.cpp"
    "tests/CommonRegexesTests.cpp"
    "tests/ConvenientParserTests.cpp"
    "tests/ConverterVersionTests.cpp"
    "tests/DateTests.cpp"
    "tests/GameVersionTests.cpp"
    "tests/iconvliteTests.cpp"
    "tests/LogTests.cpp"
    "tests/ModFilesystemTests.cpp"
    "tests/ModLoaderTests.cpp"
    "tests/ModParserTests.cpp"
    "tests/OSCompatibilityLayerTests.cpp"
    "tests/ParserHelperTests.cpp"
    "tests/ParserTests.cpp"
    "tests/StringUtilsTests.cpp"
    "tests/Localization/LocalizationBlockTests.cpp"
    "tests/Localization/LocalizationDatabaseTests.cpp"
    )
source_group("tests" FILES ${tests})

add_executable(commonItemsTests ${tests})

target_link_libraries(commonItemsTests
                      PRIVATE
                      commonLib
                      GTest::gtest
                      GTest::gtest_main
                      GTest::gmock
                      )
if (CMAKE_SYSTEM_NAME STREQUAL "Windows")
	target_compile_options(commonItemsTests PRIVATE /permissive /W4 /w14242 /w14254 /w14263 /w14265 /w14287 /we4289 /w14296 /w14311 /w14545 /w14546
	                       /w14547 /w14549 /w14555 /w14619 /w14640 /w14826 /w14905 /w14906 /w14928 /external:anglebrackets /external:W0 /WX)
elseif (CMAKE_SYSTEM_NAME STREQUAL "Linux")
endif()

file(COPY "tests/TestFiles/" DESTINATION "./")
if (CMAKE_SYSTEM_NAME STREQUAL "Linux" AND COVERAGE)
	add_custom_target(lcov COMMAND mkdir -p lcoverage)
	add_custom_command(TARGET lcov
	                   COMMAND echo "=================== LCOV ===================="
	                   COMMAND echo "-- Passing lcov tool under code coverage"
	                   COMMAND lcov --c --d CMakeFiles/commonLib.dir --o lcoverage/coverage.info --ignore-errors inconsistent,inconsistent,unused,unused
	                   COMMAND lcov --remove lcoverage/coverage.info "/usr/include/c++/*" -o lcoverage/coverage.info --ignore-errors inconsistent,inconsistent,unused,unused
	                   COMMAND echo "-- Generating HTML output files"
	                   COMMAND genhtml lcoverage/coverage.info --output-directory lcoverage --ignore-errors inconsistent,inconsistent,unused,unused
	                   )
	set_property(DIRECTORY APPEND PROPERTY ADDITIONAL_MAKE_CLEAN_FILES gcoverage)
	add_custom_target(init
	                  COMMAND ${CMAKE_MAKE_PROGRAM} clean
	                  COMMAND rm -f ${TEST_OUTPUT_DIRECTORY}/*.gcno
	                  COMMAND rm -f ${TEST_OUTPUT_DIRECTORY}/*.gcda
	                  WORKING_DIRECTORY ${CMAKE_BINARY_DIR}
	                  )
endif()
enable_testing()
add_test( NAME runUnitTests COMMAND commonItemsTests )