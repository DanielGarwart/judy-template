cmake_minimum_required(VERSION 3.28)
project(JudyTemplates)

include(CMakePushCheckState)
include(CheckCXXSourceRuns)

if(NOT DEFINED CMAKE_BUILD_TYPE)
  # set(CMAKE_BUILD_TYPE "RelWithDebInfo") #optimize, but include debug info
  set(CMAKE_BUILD_TYPE "Release" )
endif( NOT DEFINED CMAKE_BUILD_TYPE )

SET(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib )
SET(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib )
SET(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin )

if(NOT DEFINED JUDY_OPTIMIZE_FLAGS)
  if(CMAKE_COMPILER_IS_GNUCC)
    set(JUDY_OPTIMIZE_FLAGS "-march=native")
    set(JUDY_OPTIMIZE_FLAGS "${JUDY_OPTIMIZE_FLAGS} -flto")
  #elseif(MSVC)
    #  set(JUDY_OPTIMIZE_FLAGS "...") # <--- set MSVC flags here <---
  else()
    message( " !! Unrecognized compiler - no optimization flags set. Edit CMakeLists.txt or set JUDY_OPTIMIZE_FLAGS." )
    set(JUDY_OPTIMIZE_FLAGS "")
  endif(CMAKE_COMPILER_IS_GNUCC)
endif(NOT DEFINED JUDY_OPTIMIZE_FLAGS)
add_definitions(${JUDY_OPTIMIZE_FLAGS})

set(JUDYS_SOURCES src/judy.c src/judy.h)

if(CMAKE_COMPILER_IS_GNUCC)
  add_definitions( -pedantic -W -Wall -Wundef -Wfloat-equal -Wshadow -Winline -Wno-long-long )
endif(CMAKE_COMPILER_IS_GNUCC)

add_library(judy_lib STATIC ${JUDYS_SOURCES})

include_directories(src)

if(ENABLE_TESTING)
  include(CTest)
  include_directories(test)
  enable_testing()

  add_executable(pennysort test/pennySort.c test/sort.c ${JUDYS_SOURCES})
  add_executable(hexsort test/hexSort.c test/sort.c ${JUDYS_SOURCES})
  set_target_properties(pennysort hexsort PROPERTIES COMPILE_FLAGS "-DSTANDALONE")

  add_executable(judyLtest test/judyLtest.cc)
  target_link_libraries(judyLtest judy_lib)
  add_test(judyLtest ${CMAKE_BINARY_DIR}/bin/judyLtest)

  add_executable(judyL2test test/judyL2test.cc)
  target_link_libraries(judyL2test judy_lib)
  add_test(judyL2test ${CMAKE_BINARY_DIR}/bin/judyL2test)

  add_executable(judyStest test/judyStest.cc)
  target_link_libraries(judyStest judy_lib)
  add_test(judyStest ${CMAKE_BINARY_DIR}/bin/judyStest)

  add_executable(judyS2test test/judyS2test.cc)
  target_link_libraries(judyS2test judy_lib)
  add_test(judyS2test ${CMAKE_BINARY_DIR}/bin/judyS2test)
endif(ENABLE_TESTING)
