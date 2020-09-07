cmake_minimum_required(VERSION 3.12)
project(cryptopp VERSION 8.2.0)

option(BUILD_STATIC "Build static library" ON)
option(BUILD_SHARED "Build shared library" ON)
option(BUILD_TESTING "Build library tests" ON)

option(CRYPTOPP_ENABLE_SSE2 "Enable SSE2" ON)
option(CRYPTOPP_ENABLE_SSSE3 "Enable SSE3" ON)
option(CRYPTOPP_ENABLE_SSE41 "Enable SSE4.1" OFF)
option(CRYPTOPP_ENABLE_SSE42 "Enable SSE4.2" OFF)
option(CRYPTOPP_ENABLE_AVX "Enable AVX" OFF)
option(CRYPTOPP_ENABLE_AVX2 "Enable AVX2" OFF)
option(CRYPTOPP_ENABLE_AES "Enable AES" OFF)
option(CRYPTOPP_ENABLE_SHA "Enable SHA" OFF)

# Android cpu-features fix
if(ANDROID)
	file(COPY ${ANDROID_NDK}/sources/android/cpufeatures/cpu-features.h DESTINATION ${CMAKE_CURRENT_SOURCE_DIR})
endif()

### Sources

set(cryptopp_SOURCES_TEST
	test.cpp
	bench1.cpp
	bench2.cpp
	bench3.cpp
	validat0.cpp
	validat1.cpp
	validat2.cpp
	validat3.cpp
	validat4.cpp
	validat5.cpp
	validat6.cpp
	validat7.cpp
	validat8.cpp
	validat9.cpp
	validat10.cpp
	regtest1.cpp
	regtest2.cpp
	regtest3.cpp
	regtest4.cpp
	datatest.cpp
	fipsalgt.cpp
	fipstest.cpp
	dlltest.cpp
)

file(GLOB cryptopp_SOURCES "*.cpp")
list(SORT cryptopp_SOURCES)
list(REMOVE_ITEM cryptopp_SOURCES
	cryptlib.cpp
	cpu.cpp
	integer.cpp
	pch.cpp
	simple.cpp
	adhoc.cpp
	${cryptopp_SOURCES_TEST}
)
set(cryptopp_SOURCES
	cryptlib.cpp
	cpu.cpp
	integer.cpp
	${cryptopp_SOURCES}
)

# Android fix
if(ANDROID)
	list(APPEND cryptopp_SOURCES
		${ANDROID_NDK}/sources/android/cpufeatures/cpu-features.c
	)
endif()

### Targets

add_library(cryptopp-obj OBJECT ${cryptopp_SOURCES})
target_include_directories(cryptopp-obj PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})

if(WIN32)
	target_link_libraries(cryptopp-obj PRIVATE ws2_32)
endif()

find_package(Threads)
target_link_libraries(cryptopp-obj PRIVATE Threads::Threads)

if(ANDROID)
	if(${ANDROID_ABI} STREQUAL "x86")
		set(CRYPTOPP_IA32_SSE2 1)
		set(CRYPTOPP_IA32_SSSE3 1)
		set(CRYPTOPP_IA32_SSE41 0)
		set(CRYPTOPP_IA32_SSE42 0)
		set(CRYPTOPP_IA32_AES 0)
	elseif(${ANDROID_ABI} STREQUAL "x86_64")
		set(CRYPTOPP_IA32_SSE2 1)
		set(CRYPTOPP_IA32_SSSE3 1)
		set(CRYPTOPP_IA32_SSE41 1)
		set(CRYPTOPP_IA32_SSE42 1)
		set(CRYPTOPP_IA32_AES 1)
	else()
		set(CRYPTOPP_IA32_SSE2 0)
		set(CRYPTOPP_IA32_SSSE3 0)
		set(CRYPTOPP_IA32_SSE41 0)
		set(CRYPTOPP_IA32_SSE42 0)
		set(CRYPTOPP_IA32_AES 0)
	endif()

	set(CRYPTOPP_IA32_AVX 0)
	set(CRYPTOPP_IA32_AVX2 0)
	set(CRYPTOPP_IA32_SHA 0)
endif()

if(CRYPTOPP_ENABLE_SSE2)
	target_compile_options(cryptopp-obj PUBLIC -msse2)
elseif()
	target_compile_definitions(cryptopp-obj PUBLIC CRYPTOPP_DISABLE_ASM)
endif()
if(CRYPTOPP_ENABLE_SSE3)
	target_compile_options(cryptopp-obj PUBLIC -msse3)
elseif()
	target_compile_definitions(cryptopp-obj PUBLIC CRYPTOPP_DISABLE_SSE3)
endif()
if(CRYPTOPP_ENABLE_SSE41)
	target_compile_options(cryptopp-obj PUBLIC -msse4.1)
elseif()
	target_compile_definitions(cryptopp-obj PUBLIC CRYPTOPP_DISABLE_SSE4)
endif()
if(CRYPTOPP_ENABLE_SSE42)
	target_compile_options(cryptopp-obj PUBLIC -msse4.2)
elseif()
	target_compile_definitions(cryptopp-obj PUBLIC CRYPTOPP_DISABLE_SSE4)
endif()
if(CRYPTOPP_ENABLE_AVX)
	target_compile_options(cryptopp-obj PUBLIC -mavx)
elseif()
	target_compile_definitions(cryptopp-obj PUBLIC CRYPTOPP_DISABLE_AVX)
endif()
if(CRYPTOPP_ENABLE_AVX2)
	target_compile_options(cryptopp-obj PUBLIC -mavx2)
elseif()
	target_compile_definitions(cryptopp-obj PUBLIC CRYPTOPP_DISABLE_AVX2)
endif()
if(CRYPTOPP_ENABLE_AES)
	target_compile_options(cryptopp-obj PUBLIC -maes)
elseif()
	target_compile_definitions(cryptopp-obj PUBLIC CRYPTOPP_DISABLE_AESNI)
endif()
if(CRYPTOPP_ENABLE_SHA)
	target_compile_options(cryptopp-obj PUBLIC -msha)
elseif()
	target_compile_definitions(cryptopp-obj PUBLIC CRYPTOPP_DISABLE_SHANI)
endif()

target_compile_definitions(cryptopp-obj PUBLIC CRYPTOPP_DISABLE_CLMUL)

if(BUILD_STATIC)
	add_library(cryptopp-static STATIC $<TARGET_OBJECTS:cryptopp-obj>)
	target_include_directories(cryptopp-static PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
endif()

if(BUILD_SHARED)
	add_library(cryptopp-shared SHARED $<TARGET_OBJECTS:cryptopp-obj>)
	target_include_directories(cryptopp-shared PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
endif()