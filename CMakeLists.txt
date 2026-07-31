cmake_minimum_required(VERSION 3.22)
project(PhlosionVFX VERSION 0.1.0 LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

if (CMAKE_SOURCE_DIR STREQUAL CMAKE_CURRENT_SOURCE_DIR)
    set(_phlosion_vfx_tests_default ON)
else()
    set(_phlosion_vfx_tests_default OFF)
endif()

set(_phlosion_vfx_werror_default OFF)
if (DEFINED ENV{CI} AND NOT "$ENV{CI}" STREQUAL "" AND NOT "$ENV{CI}" STREQUAL "0")
    set(_phlosion_vfx_werror_default ON)
endif()

option(PHLOSION_VFX_BUILD_TESTS
    "Build standalone Phlosion VFX contract tests"
    ${_phlosion_vfx_tests_default})
option(PHLOSION_VFX_ENABLE_WARNINGS
    "Enable compiler warnings for Phlosion VFX sources"
    ON)
option(PHLOSION_VFX_WARNINGS_AS_ERRORS
    "Treat Phlosion VFX warnings as errors"
    ${_phlosion_vfx_werror_default})

set(PHLOSION_VFX_ENGINE_GIT_REPOSITORY
    "https://github.com/AdamWentworth/PhlosionEngine.git"
    CACHE STRING
    "Phlosion Engine Git repository")
set(PHLOSION_VFX_ENGINE_GIT_TAG
    "175a69e09f5ad90b7afa4c0fdd3ce64b79de5ac9"
    CACHE STRING
    "Pinned Phlosion Engine commit")

if (NOT TARGET Phlosion::Engine)
    set(_phlosion_vfx_sibling_engine
        "${CMAKE_CURRENT_SOURCE_DIR}/../PhlosionEngine")
    set(_phlosion_vfx_engine_source_default "")
    if (EXISTS "${_phlosion_vfx_sibling_engine}/CMakeLists.txt")
        set(_phlosion_vfx_engine_source_default
            "${_phlosion_vfx_sibling_engine}")
    endif()
    set(PHLOSION_VFX_ENGINE_SOURCE_DIR
        "${_phlosion_vfx_engine_source_default}"
        CACHE PATH
        "Local Phlosion Engine checkout; empty downloads the pinned commit")

    set(PHLOSION_BUILD_TESTS OFF CACHE BOOL
        "Build Phlosion Engine tests as part of Phlosion VFX"
        FORCE)
    if (PHLOSION_VFX_ENGINE_SOURCE_DIR)
        if (NOT EXISTS "${PHLOSION_VFX_ENGINE_SOURCE_DIR}/CMakeLists.txt")
            message(FATAL_ERROR
                "PHLOSION_VFX_ENGINE_SOURCE_DIR does not contain CMakeLists.txt: "
                "${PHLOSION_VFX_ENGINE_SOURCE_DIR}")
        endif()
        add_subdirectory(
            "${PHLOSION_VFX_ENGINE_SOURCE_DIR}"
            "${CMAKE_CURRENT_BINARY_DIR}/phlosion-engine")
        message(STATUS
            "Phlosion VFX is using local Phlosion Engine: "
            "${PHLOSION_VFX_ENGINE_SOURCE_DIR}")
    else()
        include(FetchContent)
        FetchContent_Declare(
            PhlosionEngine
            GIT_REPOSITORY "${PHLOSION_VFX_ENGINE_GIT_REPOSITORY}"
            GIT_TAG "${PHLOSION_VFX_ENGINE_GIT_TAG}"
            GIT_SHALLOW FALSE
        )
        FetchContent_MakeAvailable(PhlosionEngine)
        message(STATUS
            "Phlosion VFX is using pinned Phlosion Engine commit: "
            "${PHLOSION_VFX_ENGINE_GIT_TAG}")
    endif()
endif()

if (NOT TARGET Phlosion::Engine OR NOT TARGET Phlosion::VfxPreview)
    message(FATAL_ERROR
        "Phlosion Engine did not provide the targets required by Phlosion VFX.")
endif()

set(PHLOSION_VFX_SOURCE_ROOT "${CMAKE_CURRENT_SOURCE_DIR}/src")

function(phlosion_vfx_apply_settings target_name)
    target_include_directories(${target_name}
        PUBLIC
            "$<BUILD_INTERFACE:${PHLOSION_VFX_SOURCE_ROOT}>"
    )

    if (MSVC)
        target_compile_options(${target_name}
            PRIVATE
                /permissive-
                /Zc:__cplusplus
                /FS
                /MP
        )
        if (PHLOSION_VFX_ENABLE_WARNINGS)
            target_compile_options(${target_name} PRIVATE /W4)
            if (PHLOSION_VFX_WARNINGS_AS_ERRORS)
                target_compile_options(${target_name} PRIVATE /WX)
            endif()
        endif()
    elseif (PHLOSION_VFX_ENABLE_WARNINGS)
        target_compile_options(${target_name} PRIVATE -Wall -Wextra -Wpedantic)
        if (PHLOSION_VFX_WARNINGS_AS_ERRORS)
            target_compile_options(${target_name} PRIVATE -Werror)
        endif()
    endif()
endfunction()

add_library(phlosion_vfx_core STATIC
    src/vfx/effects/shared/SharedAuthoredBatchVFX.cpp
    src/vfx/effects/scratch/ScratchGlowVFX.cpp
    src/vfx/effects/tackle/TackleSmokeVFX.cpp
    src/vfx/runtime/shared/SharedAuthoredVfxHelpers.cpp
)
target_sources(phlosion_vfx_core PRIVATE
    src/vfx/effects/growl/GrowlWaveVFX.h
    src/vfx/effects/growl/GrowlWaveVfxConfig.h
    src/vfx/effects/scratch/ScratchGlowVFX.h
    src/vfx/effects/shared/SharedAuthoredBatchVFX.h
    src/vfx/effects/tackle/TackleSmokeVFX.h
    src/vfx/runtime/shared/SharedAuthoredVfxHelpers.h
)
target_link_libraries(phlosion_vfx_core
    PUBLIC
        Phlosion::Engine
)
phlosion_vfx_apply_settings(phlosion_vfx_core)
add_library(PhlosionVFX::Core ALIAS phlosion_vfx_core)

add_library(phlosion_vfx_runtime STATIC
    src/vfx/runtime/shared/SharedAuthoredVfxSubmission.cpp
    src/vfx/runtime/shared/SharedAuthoredVfxBridge.cpp
    src/vfx/runtime/shared/SharedAuthoredVfxBatches.cpp
)
target_sources(phlosion_vfx_runtime PRIVATE
    src/vfx/runtime/shared/SharedAuthoredVfxSubmission.h
    src/vfx/runtime/shared/SharedAuthoredVfxBridge.h
    src/vfx/runtime/shared/SharedAuthoredVfxBatches.h
)
target_link_libraries(phlosion_vfx_runtime
    PUBLIC
        PhlosionVFX::Core
)
phlosion_vfx_apply_settings(phlosion_vfx_runtime)
add_library(PhlosionVFX::Runtime ALIAS phlosion_vfx_runtime)

add_library(phlosion_vfx_preview STATIC
    src/vfx/preview/VfxLibraryPreviewProject.cpp
    src/vfx/preview/effects/GrowlLabPreviewEffect.cpp
    src/vfx/preview/effects/LeerLabPreviewEffect.cpp
    src/vfx/preview/effects/ScratchLabPreviewEffect.cpp
    src/vfx/preview/effects/TackleLabPreviewEffect.cpp
    src/vfx/preview/growl/GrowlPreviewController.cpp
    src/vfx/preview/leer/LeerPreviewController.cpp
    src/vfx/preview/shared/SharedAuthoredVfxRenderer.cpp
    src/vfx/preview/scratch/ScratchPreviewController.cpp
    src/vfx/preview/tackle/TacklePreviewController.cpp
)
target_sources(phlosion_vfx_preview PRIVATE
    src/vfx/preview/VfxLibraryPreviewProject.h
    src/vfx/preview/effects/GrowlLabPreviewEffect.h
    src/vfx/preview/effects/LeerLabPreviewEffect.h
    src/vfx/preview/effects/ScratchLabPreviewEffect.h
    src/vfx/preview/effects/TackleLabPreviewEffect.h
    src/vfx/preview/growl/GrowlPreviewController.h
    src/vfx/preview/leer/LeerPreviewController.h
    src/vfx/preview/shared/ControllerBackedPreviewEffect.h
    src/vfx/preview/shared/SharedPreviewControllerBase.h
    src/vfx/preview/shared/SharedAuthoredVfxRenderer.h
    src/vfx/preview/scratch/ScratchPreviewController.h
    src/vfx/preview/tackle/TacklePreviewController.h
)
target_link_libraries(phlosion_vfx_preview
    PUBLIC
        PhlosionVFX::Runtime
        Phlosion::VfxPreview
)
phlosion_vfx_apply_settings(phlosion_vfx_preview)
add_library(PhlosionVFX::Preview ALIAS phlosion_vfx_preview)

add_library(phlosion_vfx INTERFACE)
target_link_libraries(phlosion_vfx
    INTERFACE
        PhlosionVFX::Runtime
)
add_library(PhlosionVFX::PhlosionVFX ALIAS phlosion_vfx)

if (PHLOSION_VFX_BUILD_TESTS)
    include(CTest)
    add_executable(phlosion_vfx_contract_tests
        tests/PhlosionVfxContracts.cpp
    )
    target_link_libraries(phlosion_vfx_contract_tests
        PRIVATE
            PhlosionVFX::Runtime
    )
    phlosion_vfx_apply_settings(phlosion_vfx_contract_tests)
    add_test(
        NAME phlosion_vfx_contracts
        COMMAND phlosion_vfx_contract_tests
    )
endif()
