cmake_minimum_required(VERSION 3.5)

project(libAVSForAudioEffect)

# Set version number.
set(avs_VERSION_MAJOR 3)
set(avs_VERSION_MINOR 0)
set(avs_VERSION_PATCH 8)

# C++ standard requirements.
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# 设置C语言标准
set(CMAKE_C_STANDARD 99)
set(CMAKE_C_STANDARD_REQUIRED ON)

# build for iphone or simulator
# default is iphone
if(NOT IOS_SDK)
  message(FATAL_ERROR "IOS_SDK not provided (iphone/iphonesimulator)")
endif()

if(NOT IOS_ARCHS)
  message(FATAL_ERROR "IOS_ARCHS not provided")
endif()

# Set the sdk version and the min sdk version
set(IPHONE_SDKVER "13.2")
set(IPHONE_VERSION_MIN "9.0")

# Set the sdk to build with
if("${IOS_SDK}" STREQUAL "iphone")
  set(CMAKE_OSX_SYSROOT "/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/SDKs/iPhoneOS.sdk/")
elseif("${IOS_SDK}" STREQUAL "iphonesimulator")
  set(CMAKE_OSX_SYSROOT "/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneSimulator.platform/Developer/SDKs/iPhoneSimulator.sdk")
else()
  message(FATAL_ERROR "IOS_SDK must be set to iphone/iphonesimulator")  
endif()

set(CROSS_ROOT_DIR "/Applications/Xcode.app/Contents/Developer/Toolchains/XcodeDefault.xctoolchain/usr/lib/clang/10.0.1/")
set(CMAKE_FIND_ROOT_PATH ${CROSS_ROOT_DIR})

set(CMAKE_CXX_FLAGS "-stdlib=libc++")

# Project configuration.
#set(LIBWEBRTC_INCLUDE_PATH "" CACHE STRING "libwebrtc include path")
#set(LIBWEBRTC_BINARY_PATH "" CACHE STRING "libwebrtc binary path")
set(LIBWEBRTC_INCLUDE_PATH "/Users/lco/Desktop/webrtc-checkout/src")
set(LIBWEBRTC_BINARY_PATH "/Users/lco/Desktop/webrtc-checkout/src/out_ios_libs/universal")

if (NOT LIBWEBRTC_INCLUDE_PATH)
    message(FATAL_ERROR "LIBWEBRTC_INCLUDE_PATH not provided")
endif ()

if (NOT LIBWEBRTC_BINARY_PATH)
    message(FATAL_ERROR "LIBWEBRTC_BINARY_PATH not provided")
endif ()

message("\n=========== libAVSForAudioEffect Build Configuration ===========\n")
#message(STATUS "MEDIASOUPCLIENT_BUILD_TESTS : " ${MEDIASOUPCLIENT_BUILD_TESTS})
#message(STATUS "MEDIASOUPCLIENT_LOG_TRACE   : " ${MEDIASOUPCLIENT_LOG_TRACE})
#message(STATUS "MEDIASOUPCLIENT_LOG_DEV     : " ${MEDIASOUPCLIENT_LOG_DEV})
message(STATUS "LIBWEBRTC_INCLUDE_PATH      : " ${LIBWEBRTC_INCLUDE_PATH})
message(STATUS "LIBWEBRTC_BINARY_PATH       : " ${LIBWEBRTC_BINARY_PATH})
message(STATUS "PROJECT_NAME                : " ${PROJECT_NAME})
message(STATUS "PROJECT_SOURCE_DIR          : " ${PROJECT_SOURCE_DIR})
message("")

# GCC >= 4.9 is required due to std::regex use.
if ("${CMAKE_CXX_COMPILER_ID}" STREQUAL "GNU")
    if (CMAKE_CXX_COMPILER_VERSION VERSION_LESS 4.9)
        message(FATAL_ERROR "GCC version must be at least 4.9!")
    endif ()
endif ()

set(
        SOURCE_FILES_C
        src/audio_effect/aueffect.c
        src/base/base.c
        src/dict/dict.c
        #src/msystem/device.c
        #src/msystem/msystem.c
        #src/msystem/msys_default.c
        src/queue/locked_queue.c
        src/sem/semaphore.c
        src/string/wordexp.c
        src/version/version.c
        src/log/log.c
        )

set(
        SOURCE_FILES
        src/audio_effect/chorus.cpp
        src/audio_effect/reverb.cpp
        src/audio_effect/pitch_shift.cpp
        src/audio_effect/pace_shift.cpp
        src/audio_effect/vocoder.cpp
        src/audio_effect/auto_tune.cpp
        src/audio_effect/harmonizer.cpp
        src/audio_effect/normalizer.cpp
        src/audio_effect/pitch_cycler.cpp
        src/audio_effect/pass_through.cpp
        src/audio_effect/find_pitch_lags.cpp
        src/audio_effect/time_scale.cpp
        src/audio_effect/biquad.cpp
        src/audio_effect/wav_interface.cpp
        src/audio_effect/pcm_interface.cpp
        #src/audio_io/audio_io.cpp
        #src/audio_io/mock/fake_audiodevice.cpp
        )

file(GLOB_RECURSE SOURCE_FILES_RE ${PROJECT_SOURCE_DIR}/contrib/re/src/*.c)

# include header files for the IDEs
file(GLOB_RECURSE HEADER_FILES_RE ${PROJECT_SOURCE_DIR}/contrib/re/src/*.h ${PROJECT_SOURCE_DIR}/contrib/re/include/*.h)

# Create target.
add_library(${PROJECT_NAME} STATIC ${SOURCE_FILES_C} ${SOURCE_FILES} ${SOURCE_FILES_RE} ${HEADER_FILES_RE})

message("\n=========== 33333333 ===========\n")

# Source Dependencies.
#add_subdirectory(re "${PROJECT_SOURCE_DIR}/contrib/re")

# Add some compile flags to our source files.
if (MSVC)
    message("\n=========== 66666666 ===========\n")
    set_source_files_properties(${SOURCE_FILES_C} ${SOURCE_FILES}
            PROPERTIES COMPILE_FLAGS "/W3 /WX")
else ()
    message("\n=========== 77777777 ===========\n")
    set_source_files_properties(${SOURCE_FILES_C} ${SOURCE_FILES}
            PROPERTIES COMPILE_FLAGS -Wall -Wextra -Wpedantic)
endif ()

# Private (implementation) header files.
target_include_directories(${PROJECT_NAME} PRIVATE
        "${PROJECT_SOURCE_DIR}/include"
        "${PROJECT_SOURCE_DIR}/src/audio_effect"
        "${PROJECT_SOURCE_DIR}/src/audio_io"
        "${PROJECT_SOURCE_DIR}/src/audio_io/mock"
        "${PROJECT_SOURCE_DIR}/src/msystem"
        "${PROJECT_SOURCE_DIR}/contrib/re"
        "${PROJECT_SOURCE_DIR}/contrib/re/include"
        )

# Public (interface) headers from dependencies.
target_include_directories(${PROJECT_NAME} PUBLIC
        "${LIBWEBRTC_INCLUDE_PATH}"
        "${LIBWEBRTC_INCLUDE_PATH}/rtc_base"
        "${LIBWEBRTC_INCLUDE_PATH}/third_party/abseil-cpp"
        "${LIBWEBRTC_INCLUDE_PATH}/third_party/opus/src"
        "${LIBWEBRTC_INCLUDE_PATH}/third_party/opus/src/src"
        "${LIBWEBRTC_INCLUDE_PATH}/third_party/opus/src/celt"
        "${LIBWEBRTC_INCLUDE_PATH}/third_party/opus/src/include"
        "${LIBWEBRTC_INCLUDE_PATH}/third_party/opus/src/silk"
        "${LIBWEBRTC_INCLUDE_PATH}/third_party/boringssl/src/include"
        )



# Public (interface) dependencies.
target_link_libraries(${PROJECT_NAME} PUBLIC
	${LIBWEBRTC_BINARY_PATH}/libwebrtc${CMAKE_STATIC_LIBRARY_SUFFIX}
)

# Compile definitions for libwebrtc.
target_compile_definitions(${PROJECT_NAME} PUBLIC
        $<$<NOT:$<PLATFORM_ID:Windows>>:WEBRTC_POSIX>
        $<$<PLATFORM_ID:Windows>:WEBRTC_WIN>
        $<$<PLATFORM_ID:Darwin>:WEBRTC_MAC>
        )
message("\n=========== 55555555 ===========\n")
install(TARGETS libAVSForAudioEffect DESTINATION lib)
#install(FILES ${HEADER_FILES} DESTINATION include/libAVSForAudioEffect)