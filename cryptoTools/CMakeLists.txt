project(cryptoTools LANGUAGES C CXX ASM ASM_NASM)

include(FindPkgConfig)

get_directory_property(hasParent PARENT_DIRECTORY)

# add the source files
file(GLOB_RECURSE SRCS *.cpp *.c)
if (ENABLE_SSE AND UNIX)
	list(APPEND SRCS Crypto/asm/sha_lnx.asm)
elseif (ENABLE_SSE AND WIN32)
	list(APPEND SRCS Crypto/asm/sha_win64.asm)
else()
	add_compile_definitions(NO_INTEL_ASM_SHA1)
endif()
add_library(cryptoTools STATIC ${SRCS})

# make projects that include cryptoTools use this as an include folder
target_include_directories(cryptoTools PUBLIC
                    $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/..>
                    $<INSTALL_INTERFACE:include>)
target_include_directories(cryptoTools PUBLIC
                    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/..>
                    $<INSTALL_INTERFACE:include>)

if(NOT MSVC)
    target_compile_options(cryptoTools PUBLIC $<$<COMPILE_LANGUAGE:CXX>:-std=c++17> $<$<OR:$<COMPILE_LANGUAGE:C>,$<COMPILE_LANGUAGE:CXX>>:-pthread>)
    target_link_options(cryptoTools PUBLIC -pthread)

    if(ENABLE_SSE)
        target_compile_options(cryptoTools PUBLIC $<$<OR:$<COMPILE_LANGUAGE:C>,$<COMPILE_LANGUAGE:CXX>>:-maes -msse2 -msse3 -mssse3 -msse4.1 -mpclmul>)
    endif()
    if(ENABLE_AVX)
        target_compile_options(cryptoTools PUBLIC $<$<OR:$<COMPILE_LANGUAGE:C>,$<COMPILE_LANGUAGE:CXX>>:-mavx2>)
    endif()
endif()



## Relic
###########################################################################

if (ENABLE_RELIC)
  target_link_libraries(cryptoTools relic)
endif (ENABLE_RELIC)

# libsodium
###########################################################################

if (ENABLE_SODIUM)
  target_link_libraries(cryptoTools sodium)
endif (ENABLE_SODIUM)

## WolfSSL
###########################################################################

if(ENABLE_WOLFSSL)
  target_include_directories(cryptoTools PUBLIC
                    $<BUILD_INTERFACE:${WOLFSSL_LIB_INCLUDE_DIRS}>
                    $<INSTALL_INTERFACE:>)
  target_link_libraries(cryptoTools ${WOLFSSL_LIB})
endif(ENABLE_WOLFSSL)

## Boost
###########################################################################


if(ENABLE_BOOST)
    target_include_directories(cryptoTools PUBLIC
                    $<BUILD_INTERFACE:${Boost_INCLUDE_DIR}>
                    $<INSTALL_INTERFACE:>)
    target_link_libraries(cryptoTools ${Boost_LIBRARIES})
endif()



