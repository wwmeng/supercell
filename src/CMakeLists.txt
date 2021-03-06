#set common includes
set(sc_includes ${sc_includes} ${CMAKE_CURRENT_SOURCE_DIR}/core)

find_package(OpenBabel2 REQUIRED)

set(sc_includes ${sc_includes} ${OpenBabel2_INCLUDE_DIRS})
set(sc_includes ${sc_includes} ${CMAKE_SOURCE_DIR}/libs/common_utils/)

find_package(Eigen3 REQUIRED)
set(sc_includes ${sc_includes} ${EIGEN3_INCLUDE_DIR})

find_package(LibArchive)
if( ${LibArchive_FOUND} )
  set(sc_includes ${sc_includes} ${LibArchive_INCLUDE_DIRS})
  add_definitions(-DLIBARCHIVE_ENABLED)
  set(CMAKE_REQUIRED_INCLUDES ${LibArchive_INCLUDE_DIRS})
  if(LINK_STATIC_LIBS)
    find_package(BZip2 REQUIRED)
    find_package(LibLZMA REQUIRED)
    set(CMAKE_REQUIRED_LIBRARIES ${LibArchive_LIBRARIES} ${LIBLZMA_LIBRARIES} ${BZIP2_LIBRARIES})
    set(LibArchive_LIBRARIES ${LibArchive_LIBRARIES} ${LIBLZMA_LIBRARIES} ${BZIP2_LIBRARIES})
  else()
    set(CMAKE_REQUIRED_LIBRARIES ${LibArchive_LIBRARIES})
  endif()
  include(CheckFunctionExists) 
  CHECK_FUNCTION_EXISTS(archive_write_set_format_filter_by_ext LibArchive_format_filter_by_ext)
  if( ${LibArchive_format_filter_by_ext} )
    add_definitions( -DLIBARCHIVE_PATCH_DISABLE )
  endif() 
else()
  message(WARNING "libarchive is not found. Packed output will be disabled.")
endif()

#set common sources
file(GLOB core_sources ${CMAKE_CURRENT_SOURCE_DIR}/core/*.cpp)
set(sc_sources ${WRAP_MEMCPY_SRC} ${sc_sources} ${core_sources})

set(sc_sources ${sc_sources} ${CMAKE_SOURCE_DIR}/libs/common_utils/others/string_utils.cpp
                             ${CMAKE_SOURCE_DIR}/libs/common_utils/others/rnd_utils.cpp
                             ${CMAKE_SOURCE_DIR}/libs/common_utils/cryst_tools/cryst_tools.cpp
                             ${CMAKE_SOURCE_DIR}/libs/common_utils/cryst_tools/comb_points.cpp)

set(sc_libraries ${sc_libraries} openbabel)

add_subdirectory(sc_cli)
#add_subdirectory(sc_qt)
