cmake_minimum_required (VERSION 3.18)

## Project Definition and Options ##

project(libstrophe)

set(libstrophe_xml "expat" CACHE STRING "XML parser (libxml2, expat)")
set(libstrophe_ssl "dummy" CACHE STRING "TLS provider (schannel, openssl, gnutls, dummy)")

## Source Files ##

add_dir(libstrophe src)

if (NOT libstrophe_xml STREQUAL "libxml2")
    remove_file(libstrophe src/parser_libxml2.c)
endif()
if (NOT libstrophe_xml STREQUAL "expat")
    remove_file(libstrophe src/parser_expat.c)
endif()

if (NOT libstrophe_ssl STREQUAL "schannel")
    remove_file(libstrophe src/tls_schannel.c)
endif()
if (NOT libstrophe_ssl STREQUAL "openssl")
    remove_file(libstrophe src/tls_openssl.c)
endif()
if (NOT libstrophe_ssl STREQUAL "gnutls")
    remove_file(libstrophe src/tls_gnutls.c)
endif()
if (NOT libstrophe_ssl STREQUAL "dummy")
    remove_file(libstrophe src/tls_dummy.c)
endif()

## Define libstrophe library ##

add_library(libstrophe STATIC ${libstrophe_SOURCES})


## Dependencies ##

target_include_directories(libstrophe PUBLIC .)

if (libstrophe_xml STREQUAL "libxml2")
    find_package(LibXml2 REQUIRED)
    target_include_directories(libstrophe PRIVATE ${LIBXML2_INCLUDE_DIR})
    target_link_libraries(libstrophe PRIVATE ${LIBXML2_LIBRARIES})
endif()
if (libstrophe_xml STREQUAL "expat")
    find_package(expat CONFIG REQUIRED)
    target_link_libraries(libstrophe PRIVATE expat::expat)
endif()
if (libstrophe_ssl STREQUAL "schannel")
    target_link_libraries(libstrophe PRIVATE Ws2_32)
endif()

## Properties ##

# Add .pdb for release builds #
if(CMAKE_CXX_COMPILER_ID MATCHES "MSVC" AND CMAKE_BUILD_TYPE MATCHES "Release")
   target_compile_options(libstrophe PRIVATE /Zi)
   set_target_properties(libstrophe PROPERTIES
        LINK_FLAGS "/INCREMENTAL:NO /DEBUG /OPT:REF /OPT:ICF"
        COMPILE_PDB_NAME libstrophe 
        COMPILE_PDB_OUTPUT_DIR ${CMAKE_BINARY_DIR}
   )
endif()
