cmake_minimum_required(VERSION 3.5...3.31)
set(CMAKE_PROJECT_TOP_LEVEL_INCLUDES .devcontainer/cmake/xproinc.cmake)
project(wirehair)


set(CMAKE_CXX_STANDARD 11)

set(CMAKE_CONFIGURATION_TYPES "Debug;Release" CACHE STRING "" FORCE)

if(NOT CMAKE_BUILD_TYPE)
    set(CMAKE_BUILD_TYPE Release)
endif()

option(MARCH_NATIVE "Use -march=native option" ON)
option(BUILD_TESTS "build testsuite" ON)

set(LIB_SOURCE_FILES
        wirehair.cpp
        include/wirehair/wirehair.h
        cm256.cpp
        include/wirehair/cm256.h
        gf256.cpp
        include/wirehair/gf256.h
        WirehairCodec.cpp
        WirehairCodec.h
        WirehairTools.cpp
        WirehairTools.h
        )

set(UNIT_TEST_SOURCE_FILES
        test/SiameseTools.cpp
        test/SiameseTools.h
        test/UnitTest.cpp
        )

set(GEN_SMALL_DSEEDS
        test/SiameseTools.cpp
        test/SiameseTools.h
        tables/GenerateSmallDenseSeeds.cpp
        )

set(GEN_PEEL_SEEDS
        test/SiameseTools.cpp
        test/SiameseTools.h
        tables/GeneratePeelSeeds.cpp
        )

set(GEN_MOST_DSEEDS
        test/SiameseTools.cpp
        test/SiameseTools.h
        tables/GenerateMostDenseSeeds.cpp
        )

set(GEN_DCOUNTS
        test/SiameseTools.cpp
        test/SiameseTools.h
        tables/GenerateDenseCount.cpp
        )

set(GEN_TABLES
        test/SiameseTools.cpp
        test/SiameseTools.h
        tables/TableGenerator.cpp
        tables/HeavyRowGenerator.cpp
        tables/HeavyRowGenerator.h
        )

if(MSVC)
else()
    set(flags -Wall -Wextra -mssse3)
    if(MARCH_NATIVE)
        list(APPEND flags -march=native)
    endif()
    include(CheckCXXCompilerFlag)
    foreach(flag ${flags})
        string(REPLACE "=" "_" _flag ${flag})
        check_cxx_compiler_flag("${flag}" has_wh${_flag})
        if(has_wh${_flag})
            set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} ${flag}")
        endif()
    endforeach()
    if(XP_BUILD_VERBOSE)
        message(STATUS "CMAKE_CXX_FLAGS: ${CMAKE_CXX_FLAGS}")
        message(STATUS "CMAKE_CXX_FLAGS_DEBUG: ${CMAKE_CXX_FLAGS_DEBUG}") # "-g -O0"
        message(STATUS "CMAKE_CXX_FLAGS_RELEASE: ${CMAKE_CXX_FLAGS_RELEASE}") # "-O3"
        message(STATUS "CMAKE_CXX_FLAGS_RELWITHDEBINFO: ${CMAKE_CXX_FLAGS_RELWITHDEBINFO}") # "-g -O3"
    endif()
    if (CMAKE_SYSTEM_PROCESSOR MATCHES "aarch64")
        add_definitions(-DLINUX_ARM=1)
    endif()
endif()

add_library(wirehair ${LIB_SOURCE_FILES})
set_target_properties(wirehair PROPERTIES VERSION 2)
set_target_properties(wirehair PROPERTIES SOVERSION 2)
target_include_directories(wirehair PUBLIC $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
  )

if (BUILD_TESTS)
    add_executable(unit_test ${UNIT_TEST_SOURCE_FILES})
    target_link_libraries(unit_test wirehair)

    add_executable(gen_small_dseeds ${GEN_SMALL_DSEEDS})
    target_link_libraries(gen_small_dseeds wirehair)

    add_executable(gen_peel_seeds ${GEN_PEEL_SEEDS})
    target_link_libraries(gen_peel_seeds wirehair)

    add_executable(gen_most_dseeds ${GEN_MOST_DSEEDS})
    target_link_libraries(gen_most_dseeds wirehair)

    add_executable(gen_dcounts ${GEN_DCOUNTS})
    target_link_libraries(gen_dcounts wirehair)

    add_executable(gen_tables ${GEN_TABLES})
    target_link_libraries(gen_tables wirehair)
endif()

include(GNUInstallDirs)

set(lib wirehair)
if(DEFINED XP_NAMESPACE)
  set(nameSpace NAMESPACE ${XP_NAMESPACE}::)
  string(PREPEND lib "${XP_NAMESPACE}::")
endif()
if(NOT DEFINED XP_INSTALL_CMAKEDIR)
  set(XP_INSTALL_CMAKEDIR ${CMAKE_INSTALL_DATADIR}/cmake)
endif()
set(targetsFile ${PROJECT_NAME}-targets)
xpPackageDevel(TARGETS_FILE ${targetsFile} LIBRARIES ${lib})
install(TARGETS wirehair EXPORT ${targetsFile}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
    PUBLIC_HEADER DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
install(EXPORT ${targetsFile} DESTINATION ${XP_INSTALL_CMAKEDIR} ${nameSpace})
install(DIRECTORY include/ DESTINATION ${CMAKE_INSTALL_INCLUDEDIR})
if(NOT DONT_INSTALL_PYTHON)
    install(DIRECTORY python DESTINATION ${CMAKE_INSTALL_PREFIX})
endif()
