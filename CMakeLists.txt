cmake_minimum_required (VERSION 2.6)
set(GOOBLY_VERSION_NUMBER "0.0.1")
set(CMAKE_EXPORT_COMPILE_COMMANDS 1)

enable_testing ()


# libgflags-dev libgtest-dev
find_package (GTest REQUIRED)

link_directories (
  ${PROJECT_SOURCE_DIR}/meta/tmp/grpc/libs/opt
  /usr/local/lib # must be last if locals aren't used
  )
include_directories (
  ${PROJECT_SOURCE_DIR}/meta/tmp/grpc/include
  ${PROJECT_SOURCE_DIR}/meta/tmp/grpc/third_party/protobuf/src
  ${PROJECT_SOURCE_DIR}/lib
  ${PROJECT_SOURCE_DIR}/proto
  )


# set (CPP_CLIENT_HEADERS
#   include/goobly/h1.hpp
#   )

set (PROTO_SOURCES
  ${PROJECT_SOURCE_DIR}/proto/goobly.pb.cc
  )

set (LIB_GOOBLY_SOURCES
  ${PROTO_SOURCES}
  )

function (join VALUES SEP OUTPUT)
  string (REPLACE ";" "${SEP}" _TMP_STR "${VALUES}")
  set (${OUTPUT} "${_TMP_STR}" PARENT_SCOPE)
endfunction ()


set(BASE_FLAGS
  "-fPIC"
  "-fuse-ld=gold"
  "-Wall"
  "-Werror"
  "-Wextra"
  "-Wformat"
  "-pedantic"
  "-Wmissing-braces"
  "-Wparentheses"
  "-Wpointer-arith"
  "-Wformat-security"
  "-Wunused"
  "-Wno-unused-parameter"
  "-Wcast-align"
  "-Wmissing-field-initializers"
  "-o2"
  )

set (CXX_FLAG_LIST
  ${BASE_FLAGS}
  "-std=c++1y"
  "-ftemplate-depth-256"
  "-Wnon-virtual-dtor"
  "-Woverloaded-virtual"
  "-Wsign-promo"
  "-Wreorder"
  )

join ("${CXX_FLAG_LIST}" " " CMAKE_CXX_FLAGS)
join ("${BASE_FLAGS}" " " CMAKE_C_FLAGS)
set (CMAKE_CXX_COMPILER clang++-3.6)
set (CMAKE_C_COMPILER clang-3.6)
set (PROTOC ${PROJECT_SOURCE_DIR}/meta/tmp/grpc/bins/opt/protobuf/protoc)

add_custom_command (
  OUTPUT ${PROTO_SOURCES}
  DEPENDS ${PROJECT_SOURCE_DIR}/proto/goobly.proto
  COMMAND ${PROTOC} -I${PROJECT_SOURCE_DIR}/proto --cpp_out=${PROJECT_SOURCE_DIR}/proto ${PROJECT_SOURCE_DIR}/proto/goobly.proto
  )

set (PROTOBUF_LIB_PATH ${PROJECT_SOURCE_DIR}/meta/tmp/grpc/libs/opt)
set (PROTOBUF_3RD_PARTY_LIB_PATH ${PROJECT_SOURCE_DIR}/meta/tmp/grpc/third_party)
set (GOOBLY_LIB_LIBRARIES
  :libjemalloc.so.1 # TODO(agallego) - add it to the apt.yml repo
  :${PROTOBUF_LIB_PATH}/libgpr.a
  :${PROTOBUF_LIB_PATH}/libgrpc++.a
  :${PROTOBUF_3RD_PARTY_LIB_PATH}/protobuf/src/.libs/libprotobuf.a
  pthread
  rt
  dl
  )
add_library (goobly SHARED
  ${LIB_GOOBLY_SOURCES}
  )

add_subdirectory (test)