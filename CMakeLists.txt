cmake_minimum_required(VERSION 3.6.1)
project(miniocpp)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_VERBOSE_MAKEFILE ON)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

find_package(OpenSSL REQUIRED)
find_package(nlohmann_json CONFIG REQUIRED)
find_package(pugixml CONFIG REQUIRED)
find_package(ZLIB REQUIRED)
find_package(curlpp REQUIRED)

set(minio_cpp_src
    src/args.cc
    src/baseclient.cc
    src/client.cc
    src/credentials.cc
    src/error.cc
    src/http.cc
    src/providers.cc
    src/request.cc
    src/response.cc
    src/select.cc
    src/signer.cc
    src/sse.cc
    src/types.cc
    src/utils.cc
    src/INIReader.cpp
    src/ini.c
)

add_library(miniocpp SHARED ${minio_cpp_src})

set_target_properties(miniocpp PROPERTIES
    POSITION_INDEPENDENT_CODE TRUE
)

target_include_directories(miniocpp
    PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
        $<INSTALL_INTERFACE:include>
)

target_link_libraries(miniocpp
    PUBLIC
        nlohmann_json::nlohmann_json
        pugixml
        OpenSSL::SSL
        OpenSSL::Crypto
        ZLIB::ZLIB
        curlpp
)

# -------------------------
# Install library
# -------------------------

install(TARGETS miniocpp
    EXPORT miniocpp-targets
    RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
    ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
    LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
)

# install headers
install(DIRECTORY include/
    DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)

# -------------------------
# CMake config files
# -------------------------

configure_package_config_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/miniocpp-config.cmake.in
    ${CMAKE_CURRENT_BINARY_DIR}/miniocpp-config.cmake
    INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/miniocpp
)

install(EXPORT miniocpp-targets
    FILE miniocpp-targets.cmake
    NAMESPACE miniocpp::
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/miniocpp
)

install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/miniocpp-config.cmake
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/miniocpp
)

# -------------------------
# pkg-config
# -------------------------

configure_file(
    ${CMAKE_CURRENT_SOURCE_DIR}/miniocpp.pc.in
    ${CMAKE_CURRENT_BINARY_DIR}/miniocpp.pc
    @ONLY
)

install(FILES
    ${CMAKE_CURRENT_BINARY_DIR}/miniocpp.pc
    DESTINATION ${CMAKE_INSTALL_LIBDIR}/pkgconfig
)