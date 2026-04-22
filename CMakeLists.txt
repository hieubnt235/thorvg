cmake_minimum_required(VERSION 4.0)
project(thorvg VERSION 1.0.0 LANGUAGES CXX)

set(THORVG_VERSION_STRING "${PROJECT_VERSION}")

# Options corresponding to meson_options.txt
option(THORVG_ENGINES_CPU "Enable CPU rasterizer engine" ON)
option(THORVG_ENGINES_GL "Enable GL rasterizer engine" OFF)
option(THORVG_ENGINES_WG "Enable WebGPU rasterizer engine" OFF)
option(THORVG_PARTIAL_RENDER "Enable Partial Rendering" ON)
option(THORVG_LOADERS_SVG "Enable SVG loader" ON)
option(THORVG_LOADERS_PNG "Enable PNG loader" OFF)
option(THORVG_LOADERS_JPG "Enable JPG loader" OFF)
option(THORVG_LOADERS_LOTTIE "Enable Lottie loader" ON)
option(THORVG_LOADERS_TTF "Enable TTF loader" ON)
option(THORVG_LOADERS_WEBP "Enable WEBP loader" OFF)
option(THORVG_SAVERS_GIF "Enable GIF saver" OFF)
option(THORVG_THREADS "Enable multi-threading task scheduler" ON)
option(THORVG_SIMD "Enable CPU Vectorization (SIMD)" OFF)
option(THORVG_BINDINGS_CAPI "Enable C API bindings" OFF)
option(THORVG_TOOLS_SVG2PNG "Enable svg2png tool" OFF)
option(THORVG_TOOLS_LOTTIE2GIF "Enable lottie2gif tool" OFF)
option(THORVG_TESTS "Enable building unit tests" OFF)
option(THORVG_LOG "Enable log message" OFF)
option(THORVG_STATIC "Force static linking" OFF)
option(THORVG_FILE_IO "Enable File IO calls" ON)
option(THORVG_EXTRA_LOTTIE_EXP "Enable Lottie expressions" OFF)
if (THORVG_EXTRA_LOTTIE_EXP)
    message(WARNING "Lottie expressions require JerryScript library. Not yet fully supported in CMake build.")
endif ()
option(THORVG_EXTRA_OPENMP "Enable OpenMP support" ON)
option(THORVG_EXTRA_OPENGL_ES "Use OpenGL ES variant" OFF)

# Set config.h variables based on options (default OFF)
set(THORVG_CPU_ENGINE_SUPPORT OFF)
set(THORVG_GL_ENGINE_SUPPORT OFF)
set(THORVG_WG_ENGINE_SUPPORT OFF)
set(THORVG_PARTIAL_RENDER_SUPPORT OFF)
set(THORVG_SVG_LOADER_SUPPORT OFF)
set(THORVG_PNG_LOADER_SUPPORT OFF)
set(THORVG_JPG_LOADER_SUPPORT OFF)
set(THORVG_LOTTIE_LOADER_SUPPORT OFF)
set(THORVG_TTF_LOADER_SUPPORT OFF)
set(THORVG_WEBP_LOADER_SUPPORT OFF)
set(THORVG_GIF_SAVER_SUPPORT OFF)
set(THORVG_THREAD_SUPPORT OFF)
set(THORVG_LOG_ENABLED OFF)
set(THORVG_FILE_IO_SUPPORT OFF)
set(THORVG_CAPI_BINDING_SUPPORT OFF)
set(THORVG_LOTTIE_EXPRESSIONS_SUPPORT OFF)
set(THORVG_OPENMP_SUPPORT OFF)
set(THORVG_AVX_VECTOR_SUPPORT OFF)
set(THORVG_NEON_VECTOR_SUPPORT OFF)
set(WIN32_LEAN_AND_MEAN ON)

# Define engine selection: if any engine is ON, we enable engines
set(THORVG_ENGINES_SUPPORT OFF)
if (THORVG_ENGINES_CPU OR THORVG_ENGINES_GL OR THORVG_ENGINES_WG)
    set(THORVG_ENGINES_SUPPORT ON)
endif ()

# Determine SIMD type
set(THORVG_SIMD_TYPE "none")
if (THORVG_SIMD)
    # Check for emscripten (disable native SIMD)
    if (CMAKE_SYSTEM_NAME STREQUAL "Emscripten" OR DEFINED EMSCRIPTEN)
        message(STATUS "Note: Native SIMD disabled for emscripten build")
        set(THORVG_SIMD_TYPE "none")
    elseif (CMAKE_SYSTEM_PROCESSOR MATCHES "^x86" OR CMAKE_SYSTEM_PROCESSOR MATCHES "amd64")
        set(THORVG_SIMD_TYPE "avx")
        set(THORVG_AVX_VECTOR_SUPPORT ON)
    elseif (CMAKE_SYSTEM_PROCESSOR MATCHES "^arm" OR CMAKE_SYSTEM_PROCESSOR MATCHES "^aarch64")
        set(THORVG_SIMD_TYPE "neon")
        set(THORVG_NEON_VECTOR_SUPPORT ON)
    endif ()
endif ()

# Handle dependencies between options (mirroring meson logic)
# Tools require corresponding loaders/savers
if (THORVG_TOOLS_LOTTIE2GIF)
    # lottie2gif requires GIF saver and Lottie loader
    set(THORVG_SAVERS_GIF ON)
    if (NOT THORVG_LOADERS_LOTTIE)
        message(STATUS "Enabling Lottie loader (required by lottie2gif tool)")
        set(THORVG_LOADERS_LOTTIE ON)
    endif ()
endif ()
if (THORVG_TOOLS_SVG2PNG AND NOT THORVG_LOADERS_SVG)
    message(STATUS "Enabling SVG loader (required by svg2png tool)")
    set(THORVG_LOADERS_SVG ON)
endif ()
# Lottie expressions require Lottie loader
if (THORVG_EXTRA_LOTTIE_EXP AND NOT THORVG_LOADERS_LOTTIE)
    message(WARNING "Disabling Lottie expressions (requires Lottie loader)")
    set(THORVG_EXTRA_LOTTIE_EXP OFF)
endif ()

# Set config.h variables based on options (enable where appropriate)
if (THORVG_ENGINES_CPU)
    set(THORVG_CPU_ENGINE_SUPPORT ON)
endif ()
if (THORVG_ENGINES_GL)
    set(THORVG_GL_ENGINE_SUPPORT ON)
endif ()
if (THORVG_ENGINES_WG)
    set(THORVG_WG_ENGINE_SUPPORT ON)
endif ()
if (THORVG_PARTIAL_RENDER)
    set(THORVG_PARTIAL_RENDER_SUPPORT ON)
endif ()
if (THORVG_LOADERS_SVG)
    set(THORVG_SVG_LOADER_SUPPORT ON)
endif ()
if (THORVG_LOADERS_PNG)
    set(THORVG_PNG_LOADER_SUPPORT ON)
endif ()
if (THORVG_LOADERS_JPG)
    set(THORVG_JPG_LOADER_SUPPORT ON)
endif ()
if (THORVG_LOADERS_LOTTIE)
    set(THORVG_LOTTIE_LOADER_SUPPORT ON)
endif ()
if (THORVG_LOADERS_TTF)
    set(THORVG_TTF_LOADER_SUPPORT ON)
endif ()
if (THORVG_LOADERS_WEBP)
    set(THORVG_WEBP_LOADER_SUPPORT ON)
endif ()
if (THORVG_SAVERS_GIF)
    set(THORVG_GIF_SAVER_SUPPORT ON)
endif ()
if (THORVG_THREADS)
    set(THORVG_THREAD_SUPPORT ON)
endif ()
if (THORVG_LOG)
    set(THORVG_LOG_ENABLED ON)
endif ()
if (THORVG_FILE_IO)
    set(THORVG_FILE_IO_SUPPORT ON)
endif ()
if (THORVG_BINDINGS_CAPI)
    set(THORVG_CAPI_BINDING_SUPPORT ON)
endif ()
if (THORVG_EXTRA_LOTTIE_EXP)
    set(THORVG_LOTTIE_EXPRESSIONS_SUPPORT ON)
endif ()
if (THORVG_EXTRA_OPENMP)
    find_package(OpenMP)
    if (OpenMP_FOUND)
        set(THORVG_OPENMP_SUPPORT ON)
        list(APPEND THORVG_LIB_DEPS OpenMP::OpenMP_CXX)
    else ()
        set(THORVG_EXTRA_OPENMP OFF)
        set(THORVG_OPENMP_SUPPORT OFF)

    endif ()

endif ()

# Generate config.h
configure_file(
        "${CMAKE_CURRENT_SOURCE_DIR}/config.h.in"
        "${CMAKE_CURRENT_BINARY_DIR}/config.h"
        @ONLY
)

# Set compile definitions based on static/shared
# (Will be applied via target_compile_definitions)

# Set compiler flags based on options
set(COMPILER_FLAGS)

# Emscripten detection
if (CMAKE_SYSTEM_NAME STREQUAL "Emscripten" OR DEFINED EMSCRIPTEN)
    set(EMSCRIPTEN_BUILD TRUE)
    # Disable native SIMD for emscripten (WASM SIMD would be different)
    if (THORVG_SIMD)
        message(STATUS "Note: Native SIMD disabled for emscripten build")
        set(THORVG_SIMD_TYPE "none")
        set(THORVG_AVX_VECTOR_SUPPORT OFF)
        set(THORVG_NEON_VECTOR_SUPPORT OFF)
    endif ()
else ()
    set(EMSCRIPTEN_BUILD FALSE)
endif ()

if (CMAKE_CXX_COMPILER_ID MATCHES "Clang" AND CMAKE_CXX_SIMULATE_ID STREQUAL "MSVC")
    # clang-cl
    if (THORVG_SIMD_TYPE STREQUAL "avx")
        list(APPEND COMPILER_FLAGS "/clang:-mavx")
    endif ()
    if (THORVG_SIMD_TYPE STREQUAL "neon")
        list(APPEND COMPILER_FLAGS "/clang:-mfpu=neon")
    endif ()
    list(APPEND COMPILER_FLAGS "/clang:-Wno-unknown-pragmas")

    if (NOT THORVG_LOG)
        list(APPEND COMPILER_FLAGS "/clang:-Wdouble-promotion")
    endif ()

    if (NOT CMAKE_CXX_FLAGS MATCHES "-fsanitize")
        list(APPEND COMPILER_FLAGS "/clang:-fno-math-errno" "/clang:-Woverloaded-virtual"
                "/clang:-fno-stack-protector" "/clang:-fno-unwind-tables"
                "/clang:-fno-asynchronous-unwind-tables")
    endif ()
elseif (MSVC)
    list(APPEND COMPILER_FLAGS "-DNOMINMAX")
else ()
    if (THORVG_SIMD_TYPE STREQUAL "avx")
        list(APPEND COMPILER_FLAGS "-mavx")
    endif ()
    if (THORVG_SIMD_TYPE STREQUAL "neon")
        list(APPEND COMPILER_FLAGS "-mfpu=neon")
    endif ()
    list(APPEND COMPILER_FLAGS "-Wno-unknown-pragmas")

    if (NOT THORVG_LOG)
        list(APPEND COMPILER_FLAGS "-Wdouble-promotion")
    endif ()

    if (NOT CMAKE_CXX_FLAGS MATCHES "-fsanitize")
        list(APPEND COMPILER_FLAGS "-fno-exceptions" "-fno-rtti" "-fno-stack-protector" "-fno-math-errno"
                "-fno-unwind-tables" "-fno-asynchronous-unwind-tables" "-Woverloaded-virtual")
    endif ()
endif ()

# Set common compiler flags
# (Will be applied via target_compile_options)

# Include directories
include_directories(inc)
include_directories(${CMAKE_CURRENT_BINARY_DIR})  # for config.h
include_directories(src)

# Common source files
set(COMMON_SOURCES
        src/common/tvgColor.cpp
        src/common/tvgCompressor.cpp
        src/common/tvgMath.cpp
        src/common/tvgStr.cpp
)

# Renderer source files
set(RENDERER_SOURCES
        src/renderer/tvgAccessor.cpp
        src/renderer/tvgAnimation.cpp
        src/renderer/tvgCanvas.cpp
        src/renderer/tvgFill.cpp
        src/renderer/tvgInitializer.cpp
        src/renderer/tvgLoaderMgr.cpp
        src/renderer/tvgPaint.cpp
        src/renderer/tvgPicture.cpp
        src/renderer/tvgRender.cpp
        src/renderer/tvgSaver.cpp
        src/renderer/tvgScene.cpp
        src/renderer/tvgShape.cpp
        src/renderer/tvgTaskScheduler.cpp
        src/renderer/tvgText.cpp
)

# CPU engine source files (if enabled)
set(CPU_ENGINE_SOURCES)
if (THORVG_ENGINES_CPU)
    set(CPU_ENGINE_SOURCES
            src/renderer/cpu_engine/tvgSwFill.cpp
            src/renderer/cpu_engine/tvgSwImage.cpp
            src/renderer/cpu_engine/tvgSwMath.cpp
            src/renderer/cpu_engine/tvgSwMemPool.cpp
            src/renderer/cpu_engine/tvgSwPostEffect.cpp
            src/renderer/cpu_engine/tvgSwRaster.cpp
            src/renderer/cpu_engine/tvgSwRenderer.cpp
            src/renderer/cpu_engine/tvgSwRle.cpp
            src/renderer/cpu_engine/tvgSwShape.cpp
            src/renderer/cpu_engine/tvgSwStroke.cpp
    )
endif ()

# GPU common source files (if enabled)
set(GPU_COMMON_SOURCES)
if (THORVG_ENGINES_GL OR THORVG_ENGINES_WG)
    set(GPU_COMMON_SOURCES
            src/renderer/gpu_engine/tvgGpuCommon.cpp
    )
endif ()

# GL engine source files (if enabled)
set(GL_ENGINE_SOURCES)
if (THORVG_ENGINES_GL)
    set(GL_ENGINE_SOURCES
            src/renderer/gpu_engine/gl/tvgGl.cpp
            src/renderer/gpu_engine/gl/tvgGlEffect.cpp
            src/renderer/gpu_engine/gl/tvgGlGeometry.cpp
            src/renderer/gpu_engine/gl/tvgGlGpuBuffer.cpp
            src/renderer/gpu_engine/gl/tvgGlTextureMgr.cpp
            src/renderer/gpu_engine/gl/tvgGlProgram.cpp
            src/renderer/gpu_engine/gl/tvgGlRenderer.cpp
            src/renderer/gpu_engine/gl/tvgGlSolidBatch.cpp
            src/renderer/gpu_engine/gl/tvgGlRenderPass.cpp
            src/renderer/gpu_engine/gl/tvgGlRenderTarget.cpp
            src/renderer/gpu_engine/gl/tvgGlRenderTask.cpp
            src/renderer/gpu_engine/gl/tvgGlShader.cpp
            src/renderer/gpu_engine/gl/tvgGlShaderSrc.cpp
            src/renderer/gpu_engine/gl/tvgGlTessellator.cpp
    )
endif ()

# WG engine source files (if enabled)
set(WG_ENGINE_SOURCES)
if (THORVG_ENGINES_WG)
    set(WG_ENGINE_SOURCES
            src/renderer/gpu_engine/wg/tvgWgBindGroups.cpp
            src/renderer/gpu_engine/wg/tvgWgCommon.cpp
            src/renderer/gpu_engine/wg/tvgWgCompositor.cpp
            src/renderer/gpu_engine/wg/tvgWgGeometry.cpp
            src/renderer/gpu_engine/wg/tvgWgPipelines.cpp
            src/renderer/gpu_engine/wg/tvgWgRenderData.cpp
            src/renderer/gpu_engine/wg/tvgWgRenderer.cpp
            src/renderer/gpu_engine/wg/tvgWgRenderTarget.cpp
            src/renderer/gpu_engine/wg/tvgWgRenderTask.cpp
            src/renderer/gpu_engine/wg/tvgWgShaderSrc.cpp
            src/renderer/gpu_engine/wg/tvgWgShaderTypes.cpp
            src/renderer/gpu_engine/wg/tvgWgTextureMgr.cpp
            src/renderer/gpu_engine/wg/tvgWgTessellator.cpp
    )
endif ()

# SVG loader source files (if enabled)
set(SVG_SOURCES)
if (THORVG_LOADERS_SVG)
    set(SVG_SOURCES
            src/loaders/svg/tvgSvgBuilder.cpp
            src/loaders/svg/tvgSvgCssStyle.cpp
            src/loaders/svg/tvgSvgLoader.cpp
            src/loaders/svg/tvgSvgPath.cpp
            src/loaders/svg/tvgSvgUtil.cpp
            src/loaders/svg/tvgXmlParser.cpp
    )
endif ()

# TTF loader source files (if enabled)
set(TTF_SOURCES)
if (THORVG_LOADERS_TTF)
    set(TTF_SOURCES
            src/loaders/ttf/tvgTtfLoader.cpp
            src/loaders/ttf/tvgTtfReader.cpp
    )
endif ()

# Lottie loader source files (if enabled)
set(LOTTIE_SOURCES)
if (THORVG_LOADERS_LOTTIE)
    set(LOTTIE_SOURCES
            src/loaders/lottie/tvgLottieAnimation.cpp
            src/loaders/lottie/tvgLottieBuilder.cpp
            src/loaders/lottie/tvgLottieInterpolator.cpp
            src/loaders/lottie/tvgLottieLoader.cpp
            src/loaders/lottie/tvgLottieModel.cpp
            src/loaders/lottie/tvgLottieModifier.cpp
            src/loaders/lottie/tvgLottieParser.cpp
            src/loaders/lottie/tvgLottieParserHandler.cpp
    )
    if (THORVG_EXTRA_LOTTIE_EXP)
        list(APPEND LOTTIE_SOURCES src/loaders/lottie/tvgLottieExpressions.cpp)
    endif ()
endif ()

# PNG loader source files (if enabled)
set(PNG_SOURCES)
set(PNG_INCLUDE_DIR)
if (THORVG_LOADERS_PNG)
    if (THORVG_STATIC)
        # Use internal lodepng
        set(PNG_SOURCES
                src/loaders/png/tvgLodePng.cpp
                src/loaders/png/tvgPngLoader.cpp
        )
        set(PNG_INCLUDE_DIR src/loaders/png)
    else ()
        # Try to find system libpng
        find_package(PNG)
        if (PNG_FOUND)
            # Use external loader wrapper
            set(PNG_SOURCES
                    src/loaders/external_png/tvgPngLoader.cpp
            )
            set(PNG_INCLUDE_DIR src/loaders/external_png)
            # Add PNG library dependency
            set(PNG_LIB PNG::PNG)
        else ()
            # Fallback to internal
            set(PNG_SOURCES
                    src/loaders/png/tvgLodePng.cpp
                    src/loaders/png/tvgPngLoader.cpp
            )
            set(PNG_INCLUDE_DIR src/loaders/png)
        endif ()
    endif ()
endif ()

# JPG loader source files (if enabled)
set(JPG_SOURCES)
if (THORVG_LOADERS_JPG)
    if (THORVG_STATIC)
        # Use internal jpg decoder
        set(JPG_SOURCES
                src/loaders/jpg/tvgJpgd.cpp
                src/loaders/jpg/tvgJpgLoader.cpp
        )
    else ()
        # Try to find system libjpeg
        find_package(JPEG)
        if (JPEG_FOUND)
            # Use external loader wrapper
            set(JPG_SOURCES
                    src/loaders/external_jpg/tvgJpgLoader.cpp
            )
            set(JPG_LIB JPEG::JPEG)
        else ()
            # Fallback to internal
            set(JPG_SOURCES
                    src/loaders/jpg/tvgJpgd.cpp
                    src/loaders/jpg/tvgJpgLoader.cpp
            )
        endif ()
    endif ()
endif ()

# WEBP loader source files (if enabled)
set(WEBP_SOURCES)
if (THORVG_LOADERS_WEBP)
    if (THORVG_STATIC)
        # Internal libwebp (simplified - would need full libwebp build)
        set(WEBP_SOURCES
                src/loaders/webp/tvgWebpLoader.cpp
        )
        message(WARNING "Internal libwebp build not fully implemented. Ensure libwebp is available externally.")
    else ()
        # Try to find system libwebp
        find_package(WEBP)
        if (WEBP_FOUND)
            # Use external loader wrapper
            set(WEBP_SOURCES
                    src/loaders/external_webp/tvgWebpLoader.cpp
            )
            set(WEBP_LIB WEBP::WEBP)
        else ()
            # Fallback to internal (simplified)
            set(WEBP_SOURCES
                    src/loaders/webp/tvgWebpLoader.cpp
            )
            message(WARNING "Internal libwebp build not fully implemented. Ensure libwebp is available externally.")
        endif ()
    endif ()
endif ()

# Raw loader source files (always included)
set(RAW_SOURCES
        src/loaders/raw/tvgRawLoader.cpp
)

# GIF saver source files (if enabled)
set(GIF_SOURCES)
if (THORVG_SAVERS_GIF)
    set(GIF_SOURCES
            src/savers/gif/tvgGifEncoder.cpp
            src/savers/gif/tvgGifSaver.cpp
    )
endif ()

# CAPI binding source files (if enabled)
set(CAPI_SOURCES)
if (THORVG_BINDINGS_CAPI)
    set(CAPI_SOURCES
            src/bindings/capi/tvgCapi.cpp
    )
endif ()

# Collect all source files
set(ALL_SOURCES
        ${COMMON_SOURCES}
        ${RENDERER_SOURCES}
        ${CPU_ENGINE_SOURCES}
        ${GPU_COMMON_SOURCES}
        ${GL_ENGINE_SOURCES}
        ${WG_ENGINE_SOURCES}
        ${SVG_SOURCES}
        ${TTF_SOURCES}
        ${LOTTIE_SOURCES}
        ${PNG_SOURCES}
        ${JPG_SOURCES}
        ${WEBP_SOURCES}
        ${RAW_SOURCES}
        ${GIF_SOURCES}
        ${CAPI_SOURCES}
)

# Library dependencies
set(THORVG_LIB_DEPS)


if (UNIX AND NOT APPLE AND NOT ANDROID AND NOT EMSCRIPTEN)
    find_library(PTHREAD_LIB pthread)
    if (PTHREAD_LIB)
        list(APPEND THORVG_LIB_DEPS ${PTHREAD_LIB})
    endif ()
endif ()

# Add external library dependencies
if (PNG_LIB)
    list(APPEND THORVG_LIB_DEPS ${PNG_LIB})
endif ()
if (JPG_LIB)
    list(APPEND THORVG_LIB_DEPS ${JPG_LIB})
endif ()
if (WEBP_LIB)
    list(APPEND THORVG_LIB_DEPS ${WEBP_LIB})
endif ()

# Add WG engine dependency
if (THORVG_ENGINES_WG AND NOT EMSCRIPTEN_BUILD)
    find_library(WGPU_NATIVE_LIB wgpu_native)
    if (WGPU_NATIVE_LIB)
        list(APPEND THORVG_LIB_DEPS ${WGPU_NATIVE_LIB})
    endif ()
endif ()

# Add GL engine dependency
if (THORVG_ENGINES_GL)
    find_package(OpenGL REQUIRED)
    list(APPEND THORVG_LIB_DEPS OpenGL::OpenGL)
endif ()

# Create the library
add_library(thorvg-${PROJECT_VERSION_MAJOR} ${ALL_SOURCES})

target_include_directories(thorvg-${PROJECT_VERSION_MAJOR}
        PUBLIC
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/inc>
        $<BUILD_INTERFACE:${CMAKE_BINARY_DIR}>
        PRIVATE
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/common>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/renderer>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/loaders>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/savers>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/bindings>
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/loaders/lottie/rapidjson>
)

# Add CPU engine include directory if enabled
if (THORVG_ENGINES_CPU)
    target_include_directories(thorvg-${PROJECT_VERSION_MAJOR}
            PRIVATE
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/renderer/cpu_engine>
    )
endif ()

# Add GPU engine include directories if enabled
if (THORVG_ENGINES_GL OR THORVG_ENGINES_WG)
    target_include_directories(thorvg-${PROJECT_VERSION_MAJOR}
            PRIVATE
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/renderer/gpu_engine>
    )
endif ()
if (THORVG_ENGINES_GL)
    target_include_directories(thorvg-${PROJECT_VERSION_MAJOR}
            PRIVATE
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/renderer/gpu_engine/gl>
    )
endif ()
if (THORVG_ENGINES_WG)
    target_include_directories(thorvg-${PROJECT_VERSION_MAJOR}
            PRIVATE
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/renderer/gpu_engine/wg>
    )
endif ()

# Add loader include directories if enabled
if (THORVG_LOADERS_SVG)
    target_include_directories(thorvg-${PROJECT_VERSION_MAJOR}
            PRIVATE
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/loaders/svg>
    )
endif ()
if (THORVG_LOADERS_TTF)
    target_include_directories(thorvg-${PROJECT_VERSION_MAJOR}
            PRIVATE
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/loaders/ttf>
    )
endif ()
if (THORVG_LOADERS_LOTTIE)
    target_include_directories(thorvg-${PROJECT_VERSION_MAJOR}
            PRIVATE
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/loaders/lottie>
    )
endif ()
if (THORVG_EXTRA_LOTTIE_EXP)
    target_include_directories(thorvg-${PROJECT_VERSION_MAJOR}
            PRIVATE
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/loaders/lottie/jerryscript/jerry-core/include>
    )
endif ()
if (THORVG_LOADERS_PNG AND PNG_INCLUDE_DIR)
    target_include_directories(thorvg-${PROJECT_VERSION_MAJOR}
            PRIVATE
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/${PNG_INCLUDE_DIR}>
    )
endif ()
if (THORVG_LOADERS_JPG)
    target_include_directories(thorvg-${PROJECT_VERSION_MAJOR}
            PRIVATE
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/loaders/jpg>
    )
endif ()
if (THORVG_LOADERS_WEBP)
    target_include_directories(thorvg-${PROJECT_VERSION_MAJOR}
            PRIVATE
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/loaders/webp>
    )
endif ()

# Add raw loader include directory (always included)
target_include_directories(thorvg-${PROJECT_VERSION_MAJOR}
        PRIVATE
        $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/loaders/raw>
)

# Add saver include directories if enabled
if (THORVG_SAVERS_GIF)
    target_include_directories(thorvg-${PROJECT_VERSION_MAJOR}
            PRIVATE
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/savers/gif>
    )
endif ()

# Add binding include directories if enabled
if (THORVG_BINDINGS_CAPI)
    target_include_directories(thorvg-${PROJECT_VERSION_MAJOR}
            PRIVATE
            $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/src/bindings/capi>
    )
endif ()

# Add compile definitions for static/shared library
if (THORVG_STATIC)
    target_compile_definitions(thorvg-${PROJECT_VERSION_MAJOR} PRIVATE TVG_STATIC)
else ()
    target_compile_definitions(thorvg-${PROJECT_VERSION_MAJOR} PRIVATE TVG_EXPORT TVG_BUILD)
endif ()

# Add GL engine compile definitions
if (THORVG_ENGINES_GL)
    if (THORVG_EXTRA_OPENGL_ES OR EMSCRIPTEN_BUILD)
        target_compile_definitions(thorvg-${PROJECT_VERSION_MAJOR} PRIVATE THORVG_GL_TARGET_GLES=1)
    else ()
        target_compile_definitions(thorvg-${PROJECT_VERSION_MAJOR} PRIVATE THORVG_GL_TARGET_GL=1)
    endif ()
endif ()

target_compile_options(thorvg-${PROJECT_VERSION_MAJOR} PRIVATE ${COMPILER_FLAGS})
target_link_libraries(thorvg-${PROJECT_VERSION_MAJOR} ${THORVG_LIB_DEPS})

if (NOT WIN32)
    set_target_properties(thorvg-${PROJECT_VERSION_MAJOR} PROPERTIES
            VERSION ${PROJECT_VERSION}
            SOVERSION ${PROJECT_VERSION_MAJOR}
    )
endif ()

install(TARGETS thorvg-${PROJECT_VERSION_MAJOR}
        LIBRARY DESTINATION lib
        ARCHIVE DESTINATION lib
        RUNTIME DESTINATION bin
)

install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/inc/
        DESTINATION include/thorvg-${PROJECT_VERSION_MAJOR}
        FILES_MATCHING PATTERN "*.h"
)

# Install additional headers
if (THORVG_LOADERS_LOTTIE)
    install(FILES src/loaders/lottie/thorvg_lottie.h
            DESTINATION include/thorvg-${PROJECT_VERSION_MAJOR}
    )
endif ()

if (THORVG_BINDINGS_CAPI)
    install(FILES src/bindings/capi/thorvg_capi.h
            DESTINATION include/thorvg-${PROJECT_VERSION_MAJOR}
    )
endif ()

# Create pkg-config file (simplified)
configure_file(
        "${CMAKE_CURRENT_SOURCE_DIR}/thorvg.pc.in"
        "${CMAKE_BINARY_DIR}/thorvg-${PROJECT_VERSION_MAJOR}.pc"
        @ONLY
)

install(FILES "${CMAKE_BINARY_DIR}/thorvg-${PROJECT_VERSION_MAJOR}.pc"
        DESTINATION lib/pkgconfig
)

# Tools
if (THORVG_TOOLS_SVG2PNG)
    add_executable(svg2png
            tools/svg2png/svg2png.cpp
            tools/svg2png/lodepng.cpp
    )
    target_link_libraries(svg2png thorvg-${PROJECT_VERSION_MAJOR})

    install(TARGETS svg2png
            RUNTIME DESTINATION bin
    )
endif ()

if (THORVG_TOOLS_LOTTIE2GIF)
    add_executable(lottie2gif
            tools/lottie2gif/lottie2gif.cpp
    )
    target_link_libraries(lottie2gif thorvg-${PROJECT_VERSION_MAJOR})

    install(TARGETS lottie2gif
            RUNTIME DESTINATION bin
    )
endif ()

# Tests
if (THORVG_TESTS)
    # Test source files
    set(TEST_SOURCES
            test/testAccessor.cpp
            test/testAnimation.cpp
            test/testFill.cpp
            test/testInitializer.cpp
            test/testLottie.cpp
            test/testMain.cpp
            test/testPaint.cpp
            test/testPicture.cpp
            test/testSavers.cpp
            test/testScene.cpp
            test/testShape.cpp
            test/testSwCanvas.cpp
            test/testSwEngine.cpp
            test/testText.cpp
    )

    # Test compiler flags
    set(TEST_COMPILER_FLAGS ${COMPILER_FLAGS})

    # Add test directory definition (as in meson build)
    # (Will be applied via target_compile_definitions)

    add_executable(tvgUnitTests ${TEST_SOURCES})

    target_include_directories(tvgUnitTests
            PRIVATE
            ${CMAKE_CURRENT_SOURCE_DIR}/test
    )

    target_compile_options(tvgUnitTests PRIVATE ${TEST_COMPILER_FLAGS})
    target_compile_definitions(tvgUnitTests PRIVATE TEST_DIR="${CMAKE_CURRENT_SOURCE_DIR}/test/resources")
    if (THORVG_STATIC)
        target_compile_definitions(tvgUnitTests PRIVATE TVG_STATIC)
    endif ()
    target_link_libraries(tvgUnitTests thorvg-${PROJECT_VERSION_MAJOR})

    # Add framework dependencies for macOS
    if (APPLE)
        target_link_libraries(tvgUnitTests "-framework Cocoa" "-framework IOKit")
    endif ()

    # Add test target
    add_test(NAME UnitTests COMMAND tvgUnitTests --success)
endif ()

# Determine OpenGL variant for summary (matching meson)
set(GL_VARIANT "None")
if (THORVG_ENGINES_GL)
    if (EMSCRIPTEN_BUILD OR THORVG_EXTRA_OPENGL_ES)
        set(GL_VARIANT "OpenGL ES")
    else ()
        set(GL_VARIANT "OpenGL")
    endif ()
endif ()

# Summary
message(STATUS "ThorVG Configuration Summary:")
message(STATUS "  Build Type: ${CMAKE_BUILD_TYPE}")
message(STATUS "  Prefix: ${CMAKE_INSTALL_PREFIX}")
message(STATUS "  File I/O: ${THORVG_FILE_IO}")
message(STATUS "  Multi-Tasking: ${THORVG_THREADS}")
message(STATUS "  Partial Rendering: ${THORVG_PARTIAL_RENDER}")
message(STATUS "  SIMD Instruction: ${THORVG_SIMD_TYPE}")
message(STATUS "  Log Message: ${THORVG_LOG}")
message(STATUS "  Tests: ${THORVG_TESTS}")
message(STATUS "Raster Engine:")
message(STATUS "  CPU: ${THORVG_ENGINES_CPU}")
message(STATUS "  GL: ${THORVG_ENGINES_GL}")
message(STATUS "  WebGPU: ${THORVG_ENGINES_WG}")
message(STATUS "Loader:")
message(STATUS "  SVG: ${THORVG_LOADERS_SVG}")
message(STATUS "  TTF: ${THORVG_LOADERS_TTF}")
message(STATUS "  LOTTIE: ${THORVG_LOADERS_LOTTIE}")
message(STATUS "  PNG: ${THORVG_LOADERS_PNG}")
message(STATUS "  JPG: ${THORVG_LOADERS_JPG}")
message(STATUS "  WEBP: ${THORVG_LOADERS_WEBP}")
message(STATUS "Saver:")
message(STATUS "  GIF: ${THORVG_SAVERS_GIF}")
message(STATUS "Binding:")
message(STATUS "  CAPI: ${THORVG_BINDINGS_CAPI}")
message(STATUS "Tool:")
message(STATUS "  Svg2Png: ${THORVG_TOOLS_SVG2PNG}")
message(STATUS "  Lottie2Gif: ${THORVG_TOOLS_LOTTIE2GIF}")
message(STATUS "Extra:")
message(STATUS "  Lottie Expressions: ${THORVG_EXTRA_LOTTIE_EXP}")
message(STATUS "  OpenMP: ${THORVG_EXTRA_OPENMP}")
message(STATUS "  OpenGL Variant: ${GL_VARIANT}")