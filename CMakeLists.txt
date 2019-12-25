cmake_minimum_required(VERSION 3.12)
project(bgfx)
set(MYROOTDIR ${CMAKE_CURRENT_SOURCE_DIR})
add_subdirectory(external)

set(BGFX_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/external/bgfx)
set(BX_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/external/bx)
set(BIMG_ROOT ${CMAKE_CURRENT_SOURCE_DIR}/external/bimg)

if (WIN32)
  set(STDC_MACROS
    __STDC_LIMIT_MACROS
    __STDC_FORMAT_MACROS
    __STDC_CONSTANT_MACROS
  )
else()
  set(STDC_MACROS )
endif()

if (APPLE)
  set(CMAKE_CXX_STANDARD 14)
  set(CMAKE_CXX_STANDARD_REQUIRED ON)
  set(CMAKE_CXX_EXTENSIONS OFF)
endif()

# BX #######################################################
file(GLOB BX_SRC
  ${BX_ROOT}/src/*.h
  ${BX_ROOT}/src/*.cpp
  ${BX_ROOT}/include/*.h
)

if (WIN32)
  file(GLOB BX_SRC_COMPAT
    ${BX_ROOT}/include/compat/msvc/*.h
    ${BX_ROOT}/include/compat/msvc/pre1600/*.h
  )
elseif(APPLE)
  file(GLOB BX_SRC_COMPAT ${BX_ROOT}/include/compat/osx/*.h)
endif()

set(BX_SRC ${BX_SRC} ${BX_SRC_COMPAT})

list(REMOVE_ITEM BX_SRC ${BX_ROOT}/src/amalgamated.cpp)

add_library(bx STATIC ${BX_SRC})
target_include_directories(bx PUBLIC ${BX_ROOT}/include)

if (WIN32)
  target_include_directories(bx PUBLIC ${BX_ROOT}/include/compat/msvc)
elseif(APPLE)
  target_include_directories(bx PUBLIC ${BX_ROOT}/include/compat/osx)
endif()

target_include_directories(bx PRIVATE ${BX_ROOT}/3rdparty)
target_compile_definitions(bx PUBLIC ${STDC_MACROS})


# BIMG #######################################################
file(GLOB BIMG_SRC
  ${BIMG_ROOT}/src/*.h
  ${BIMG_ROOT}/3rdparty/astc-codec/src/decoder/*.h
  ${BIMG_ROOT}/3rdparty/astc-codec/src/decoder/*.cc
)
add_library(bimg STATIC ${BIMG_SRC}
  ${BIMG_ROOT}/src/image.cpp
  ${BIMG_ROOT}/src/image_gnf.cpp
)
target_include_directories(bimg PUBLIC 
  ${BIMG_ROOT}/include
  ${BIMG_ROOT}/3rdparty/astc-codec
  ${BIMG_ROOT}/3rdparty/astc-codec/include
)
target_link_libraries(bimg PRIVATE bx)
target_compile_definitions(bimg PUBLIC ${STDC_MACROS})

add_library(bimg_decode STATIC ${BIMG_ROOT}/src/image_decode.cpp)
target_include_directories(bimg_decode PUBLIC 
  ${BIMG_ROOT}/include
  ${BIMG_ROOT}/3rdparty
)
target_compile_definitions(bimg_decode PUBLIC ${STDC_MACROS})
target_link_libraries(bimg_decode PRIVATE bx)

# BGFX #######################################################
file(GLOB BGFX_SRC
  ${BGFX_ROOT}/src/*.h
  ${BGFX_ROOT}/src/*.cpp
)

list(REMOVE_ITEM BGFX_SRC 
  ${BGFX_ROOT}/src/amalgamated.cpp
)

add_library(bgfx STATIC 
  ${BGFX_SRC}
)

target_include_directories(bgfx PUBLIC 
  ${BGFX_ROOT}/3rdparty
  ${BGFX_ROOT}/3rdparty/dxsdk/include
  ${BGFX_ROOT}/3rdparty/khronos
  ${BGFX_ROOT}/include
)

target_include_directories(bgfx PRIVATE ${BGFX_ROOT}/src)

target_compile_definitions(bgfx PUBLIC ${STDC_MACROS})
target_link_libraries(bgfx PRIVATE bx bimg bimg_decode)
