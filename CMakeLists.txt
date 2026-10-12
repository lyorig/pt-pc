cmake_minimum_required(VERSION 3.28)

# macOS on Apple silicon (docs/macos.md): <format> needs 13.3, every Apple silicon Mac runs 14. Left empty (a build folder
# configured before this line) the build would run only on the macOS it was built on; other hosts do not get the cache entry
if(CMAKE_HOST_APPLE AND NOT CMAKE_OSX_DEPLOYMENT_TARGET)
    set(CMAKE_OSX_DEPLOYMENT_TARGET
        "14.0"
        CACHE STRING
        "Oldest macOS the build runs on"
        FORCE
    )
endif()

# OBJCXX on macOS for the Apple MetalFX upscaler (cmake/MacOS.cmake, metalfx_backend.mm); the language is set here so its
# variables are initialized, as enable_language after project does not
if(CMAKE_HOST_APPLE OR APPLE)
    project(pt_port LANGUAGES C CXX OBJCXX)
else()
    project(pt_port LANGUAGES C CXX)
endif()

list(APPEND CMAKE_MODULE_PATH "${CMAKE_CURRENT_SOURCE_DIR}/cmake")

set(CMAKE_MSVC_RUNTIME_LIBRARY "MultiThreaded$<$<CONFIG:Debug>:Debug>")

if(CMAKE_SYSTEM_NAME STREQUAL "Linux")
    # one binary for many distributions: the C++ runtime goes into the executable, only glibc is taken from the system
    string(APPEND CMAKE_EXE_LINKER_FLAGS " -static-libstdc++ -static-libgcc")
    # and into the voice recognizer's libwhisper.so and libggml*.so (cmake/Dependencies.cmake)
    # with their copies of the C++ runtime kept private (--exclude-libs): exported, the copies in libggml.so and libwhisper.so
    # bound to each other and to the system's libstdc++ (which Mesa loads into the game), and the speech model's std::regex
    # crashed in a facet of the wrong copy on SteamOS; -Bsymbolic binds the template code each library has to its own copy too
    string(
        APPEND CMAKE_SHARED_LINKER_FLAGS
        " -static-libstdc++ -static-libgcc -Wl,--exclude-libs,ALL -Wl,-Bsymbolic"
    )
    string(
        APPEND CMAKE_MODULE_LINKER_FLAGS
        " -static-libstdc++ -static-libgcc -Wl,--exclude-libs,ALL -Wl,-Bsymbolic"
    )
endif()

include(Dependencies)

set(PT_ENGINE_SOURCES
    src/engine/anim/anim_codec.cpp
    src/engine/anim/anim_test.cpp
    src/engine/anim/demo_file.cpp
    src/engine/anim/gani.cpp
    src/engine/anim/help_bones.cpp
    src/engine/anim/motion_data.cpp
    src/engine/anim/rig.cpp
    src/engine/anim/sim_bones.cpp
    src/engine/anim/sim_physics.cpp
    src/engine/anim/skeleton.cpp
    src/engine/assets/enhanced_textures.cpp
    src/engine/assets/fmdl.cpp
    src/engine/assets/ftex.cpp
    src/engine/assets/geom.cpp
    src/engine/assets/gltf_model.cpp
    src/engine/assets/texture_cache.cpp
    src/engine/audio/audio_engine.cpp
    src/engine/audio/controller_haptics.cpp
    src/engine/audio/controller_pcm_capture.cpp
    src/engine/audio/dsp.cpp
    src/engine/audio/motion_generator.cpp
    src/engine/audio/sound_engine.cpp
    src/engine/audio/sound_package.cpp
    src/engine/audio/sound_system.cpp
    src/engine/audio/subtitles.cpp
    src/engine/audio/wem.cpp
    src/engine/audio/wwise_bank.cpp
    src/engine/audio/wwise_vorbis.cpp
    src/engine/core/crash_report.cpp
    src/engine/core/crash_upload.cpp
    src/engine/core/json.cpp
    src/engine/core/log.cpp
    src/engine/core/memory_status.cpp
    src/engine/core/pathcode.cpp
    src/engine/core/resource_path.cpp
    src/engine/core/strcode.cpp
    src/engine/data/fox2.cpp
    src/engine/fs/fox_crypt.cpp
    src/engine/fs/fpk.cpp
    src/engine/fs/mod_level.cpp
    src/engine/fs/mods.cpp
    src/engine/fs/psarc.cpp
    src/engine/fs/qar.cpp
    src/engine/fs/vfs.cpp
    src/engine/physics/collision_world.cpp
    src/engine/platform/controller_feedback.cpp
    src/engine/platform/controller_speaker.cpp
    src/engine/platform/controller_speaker_route.cpp
    src/engine/platform/graphics_presets.cpp
    src/engine/platform/http.cpp
    src/engine/platform/input.cpp
    src/engine/platform/livesplit.cpp
    src/engine/platform/os.cpp
    src/engine/platform/sdl_diag.cpp
    src/engine/platform/settings.cpp
    src/engine/platform/update_check.cpp
    src/engine/platform/user_data.cpp
    src/engine/platform/virtual_pad.cpp
    src/engine/render/light_cull.cpp
    src/engine/render/mesh.cpp
    src/engine/render/model_cache.cpp
    src/engine/render/pipeline_cache_runtime.cpp
    src/engine/render/pipeline_cache_store.cpp
    src/engine/render/raytracing.cpp
    src/engine/render/render_util.cpp
    src/engine/render/renderer.cpp
    src/engine/render/scene_frame.cpp
    src/engine/render/scene_lighting.cpp
    src/engine/render/scene_post.cpp
    src/engine/render/scene_renderer.cpp
    src/engine/render/subsurface_pass.cpp
    src/engine/render/texture_manager.cpp
    src/engine/render/upscale/dlss_backend.cpp
    src/engine/render/upscale/frame_generation.cpp
    src/engine/render/upscale/fsr_backend.cpp
    src/engine/render/upscale/metalfx_stub.cpp
    src/engine/render/upscale/scene_upscale.cpp
    src/engine/render/upscale/streamline.cpp
    src/engine/render/upscale/upscale.cpp
    src/engine/render/upscale/upscale_platform.cpp
    src/engine/render/upscale/xess_backend.cpp
    src/engine/render/vfx_pass.cpp
    src/engine/render/vk.cpp
    src/engine/render/vk_context.cpp
    src/engine/script/lua_vm.cpp
    src/engine/script/mod_lua.cpp
    src/engine/ui/asset_browser.cpp
    src/engine/ui/ffnt.cpp
    src/engine/ui/lang_file.cpp
    src/engine/ui/text_layout.cpp
    src/engine/ui/ui_batch.cpp
    src/engine/ui/uia.cpp
    src/engine/ui/uif.cpp
    src/engine/ui/uigb.cpp
    src/engine/ui/uilb.cpp
    src/engine/ui/unicode_font.cpp
    src/engine/ui/unicode_font_harfbuzz.cpp
    src/engine/vfx/vfx_effect.cpp
    src/engine/vfx/vfx_file.cpp
    src/engine/vfx/vfx_lensflare.cpp
    src/engine/vfx/vfx_schema.cpp
    src/engine/vfx/vfx_system.cpp
    src/engine/voice/microphone.cpp
    src/engine/voice/voice_recognizer.cpp
    src/engine/xr/xr_host.cpp
)
set(PT_ENGINE_HEADERS
    src/engine/anim/anim_codec.h
    src/engine/anim/anim_test.h
    src/engine/anim/demo_file.h
    src/engine/anim/gani.h
    src/engine/anim/help_bones.h
    src/engine/anim/motion_data.h
    src/engine/anim/rig.h
    src/engine/anim/sim_bones.h
    src/engine/anim/sim_physics.h
    src/engine/anim/skeleton.h
    src/engine/assets/enhanced_textures.h
    src/engine/assets/fmdl.h
    src/engine/assets/ftex.h
    src/engine/assets/geom.h
    src/engine/assets/gltf_model.h
    src/engine/assets/texture_cache.h
    src/engine/audio/audio_engine.h
    src/engine/audio/channel_layout.h
    src/engine/audio/controller_haptics.h
    src/engine/audio/controller_pcm_capture.h
    src/engine/audio/dsp.h
    src/engine/audio/motion_generator.h
    src/engine/audio/sound_engine.h
    src/engine/audio/sound_package.h
    src/engine/audio/sound_system.h
    src/engine/audio/subtitles.h
    src/engine/audio/wem.h
    src/engine/audio/wwise_bank.h
    src/engine/audio/wwise_vorbis.h
    src/engine/core/arabic_text.h
    src/engine/core/chinese_text.h
    src/engine/core/crash_report.h
    src/engine/core/crash_upload.h
    src/engine/core/czech_text.h
    src/engine/core/french_text.h
    src/engine/core/german_text.h
    src/engine/core/italian_text.h
    src/engine/core/japanese_text.h
    src/engine/core/json.h
    src/engine/core/localized_text.h
    src/engine/core/log.h
    src/engine/core/memory_status.h
    src/engine/core/parallel_for.h
    src/engine/core/pathcode.h
    src/engine/core/polish_text.h
    src/engine/core/portuguese_text.h
    src/engine/core/resource_path.h
    src/engine/core/russian_text.h
    src/engine/core/spanish_text.h
    src/engine/core/strcode.h
    src/engine/core/turkish_text.h
    src/engine/core/ukrainian_text.h
    src/engine/data/fox2.h
    src/engine/fs/fox_crypt.h
    src/engine/fs/fpk.h
    src/engine/fs/mods.h
    src/engine/fs/psarc.h
    src/engine/fs/qar.h
    src/engine/fs/vfs.h
    src/engine/physics/collision_world.h
    src/engine/platform/controller_feedback.h
    src/engine/platform/controller_speaker.h
    src/engine/platform/display_modes.h
    src/engine/platform/graphics_presets.h
    src/engine/platform/http.h
    src/engine/platform/input.h
    src/engine/platform/livesplit.h
    src/engine/platform/os.h
    src/engine/platform/sdl_diag.h
    src/engine/platform/self_integrity.h
    src/engine/platform/settings.h
    src/engine/platform/update_check.h
    src/engine/platform/user_data.h
    src/engine/platform/virtual_pad.h
    src/engine/render/camera.h
    src/engine/render/dominant_light.h
    src/engine/render/free_camera.h
    src/engine/render/gpu_types.h
    src/engine/render/hdr_output.h
    src/engine/render/light_cull.h
    src/engine/render/mesh.h
    src/engine/render/model_cache.h
    src/engine/render/pipeline_cache_store.h
    src/engine/render/raytracing.h
    src/engine/render/render_util.h
    src/engine/render/renderer.h
    src/engine/render/scene_lighting.h
    src/engine/render/scene_renderer.h
    src/engine/render/subsurface_pass.h
    src/engine/render/texture_manager.h
    src/engine/render/upscale/frame_generation.h
    src/engine/render/upscale/streamline.h
    src/engine/render/upscale/upscale.h
    src/engine/render/upscale/upscale_platform.h
    src/engine/render/vfx_pass.h
    src/engine/render/vk.h
    src/engine/render/vk_context.h
    src/engine/script/lua_vm.h
    src/engine/script/mod_lua.h
    src/engine/ui/asset_browser.h
    src/engine/ui/ffnt.h
    src/engine/ui/lang_file.h
    src/engine/ui/text_layout.h
    src/engine/ui/ui_batch.h
    src/engine/ui/uia.h
    src/engine/ui/uif.h
    src/engine/ui/uigb.h
    src/engine/ui/uilb.h
    src/engine/vfx/vfx_effect.h
    src/engine/vfx/vfx_file.h
    src/engine/vfx/vfx_lensflare.h
    src/engine/vfx/vfx_schema.h
    src/engine/vfx/vfx_system.h
    src/engine/voice/microphone.h
    src/engine/voice/voice_recognizer.h
    src/engine/xr/xr_host.h
    src/engine/xr/xr_view.h
)

set(PT_GAME_SOURCES
    src/game/archive.cpp
    src/game/archive_theater.cpp
    src/game/debug_panel.cpp
    src/game/demo_stream.cpp
    src/game/demo_system.cpp
    src/game/floor_level.cpp
    src/game/game.cpp
    src/game/game_controller.cpp
    src/game/game_objects.cpp
    src/game/game_plus_arm.cpp
    src/game/game_sound.cpp
    src/game/gimmick_animation.cpp
    src/game/input_script.cpp
    src/game/lua_entity.cpp
    src/game/message_system.cpp
    src/game/nazo.cpp
    src/game/parameters.cpp
    src/game/player.cpp
    src/game/player_animation.cpp
    src/game/prompt_textures.cpp
    src/game/render_scene.cpp
    src/game/save_data.cpp
    src/game/screen_effects.cpp
    src/game/script_api.cpp
    src/game/script_bindings.cpp
    src/game/script_host.cpp
    src/game/speedrun.cpp
    src/game/stage_data.cpp
    src/game/stage_manager.cpp
    src/game/trap_system.cpp
    src/game/ui/archive_view.cpp
    src/game/ui/demo_ui.cpp
    src/game/ui/game_ui.cpp
    src/game/ui/options_menu.cpp
    src/game/ui/pc_settings.cpp
    src/game/ui/pc_settings_page.cpp
    src/game/ui/photo_panel.cpp
    src/game/ui/photo_preset.cpp
    src/game/ui/port_credits.cpp
    src/game/ui/save_icon.cpp
    src/game/ui/subliminal.cpp
    src/game/ui/subtitle_player.cpp
    src/game/ui/ui_assets.cpp
    src/game/ui/ui_icons.cpp
    src/game/ui/uia_player.cpp
    src/game/ui/uif_view.cpp
    src/game/vfx_scene.cpp
    src/game/vr_play.cpp
    src/game/archive.h
    src/game/archive_model_camera.h
    src/game/archive_theater.h
    src/game/blur_sway.h
    src/game/debug_panel.h
    src/game/demo_stream.h
    src/game/demo_system.h
    src/game/floor_level.h
    src/game/follower_shadow.h
    src/game/game.h
    src/game/game_controller.h
    src/game/game_objects.h
    src/game/game_plus_arm.h
    src/game/game_sound.h
    src/game/gimmick_animation.h
    src/game/input_script.h
    src/game/loop_browser.h
    src/game/lua_entity.h
    src/game/message_system.h
    src/game/nazo.h
    src/game/outro_input.h
    src/game/parameters.h
    src/game/player.h
    src/game/player_animation.h
    src/game/prompt_textures.h
    src/game/render_mouse.h
    src/game/render_scene.h
    src/game/save_data.h
    src/game/screen_effects.h
    src/game/script_api.h
    src/game/script_host.h
    src/game/speedrun.h
    src/game/stage_data.h
    src/game/stage_manager.h
    src/game/trap_system.h
    src/game/ui/archive_view.h
    src/game/ui/demo_ui.h
    src/game/ui/game_ui.h
    src/game/ui/museum_layout.h
    src/game/ui/options_menu.h
    src/game/ui/pc_settings.h
    src/game/ui/pc_settings_page.h
    src/game/ui/photo_panel.h
    src/game/ui/photo_preset.h
    src/game/ui/port_credits.h
    src/game/ui/save_icon.h
    src/game/ui/subliminal.h
    src/game/ui/subtitle_player.h
    src/game/ui/ui_assets.h
    src/game/ui/ui_icons.h
    src/game/ui/uia_player.h
    src/game/ui/uif_view.h
    src/game/vfx_scene.h
    src/game/vr_play.h
    # no sources in src/decomp
    # no headers in src/decomp
)

# The release version and metadata endpoint shared by the game and installers (docs/updates.md).
# GitHub supplies release metadata and platform asset URLs; no separate JSON release asset is needed.
# This build's version: raised here for each release. tools/ci/release.py --version sets it through the environment at build
# time (cmake/version.cmake); PT_VERSION_OVERRIDE sets it for one build folder.
set(PT_VERSION_OVERRIDE
    ""
    CACHE STRING
    "Version for this build folder instead of the one in CMakeLists.txt"
)

if(PT_VERSION_OVERRIDE)
    set(PT_VERSION "${PT_VERSION_OVERRIDE}")
else()
    set(PT_VERSION "1.0.3")
endif()

# not a cache variable: an old build folder must not keep a stale address (PT_UPDATE_MANIFEST_URL_OVERRIDE sets one for a build folder)
set(PT_UPDATE_MANIFEST_URL_OVERRIDE
    ""
    CACHE STRING
    "Update metadata endpoint, instead of the GitHub Releases API"
)

set(PT_UPDATE_MANIFEST_URL
    "https://api.github.com/repos/LoreanXavier/pt-pc/releases/latest"
)

if(PT_UPDATE_MANIFEST_URL_OVERRIDE)
    set(PT_UPDATE_MANIFEST_URL "${PT_UPDATE_MANIFEST_URL_OVERRIDE}")
endif()

add_custom_target(
    pt_version
    COMMAND
        "${CMAKE_COMMAND}"
        "-DOUT=${CMAKE_CURRENT_BINARY_DIR}/generated/pt_version.h"
        "-DPT_VERSION=${PT_VERSION}"
        "-DPT_UPDATE_MANIFEST_URL=${PT_UPDATE_MANIFEST_URL}" -P
        "${CMAKE_CURRENT_SOURCE_DIR}/cmake/version.cmake"
    BYPRODUCTS "${CMAKE_CURRENT_BINARY_DIR}/generated/pt_version.h"
)

pt_add_library(pt_engine STATIC ${PT_ENGINE_SOURCES} ${PT_GAME_SOURCES})
target_sources(
    pt_engine
    PRIVATE ${PT_ENGINE_SOURCES} ${PT_GAME_SOURCES}
    PUBLIC
        FILE_SET HEADERS
            BASE_DIRS src/engine src/decomp
            FILES ${PT_ENGINE_HEADERS}
)

# Game+ (docs/gameplay.md, Game+): the content a finished game unlocks, ON since 1.0 (every release ships it). OFF builds keep
# it inert (the save still stores the finish tag and count, nothing else changes). The first configure takes the default from
# the environment variable PT_GAMEPLUS.
if(DEFINED ENV{PT_GAMEPLUS})
    set(PT_GAMEPLUS_DEFAULT "$ENV{PT_GAMEPLUS}")
else()
    set(PT_GAMEPLUS_DEFAULT ON)
endif()

option(PT_GAMEPLUS "Game+ content after a finished game" ${PT_GAMEPLUS_DEFAULT})

if(PT_GAMEPLUS)
    target_compile_definitions(pt_engine PUBLIC PT_GAMEPLUS=1)
else()
    target_compile_definitions(pt_engine PUBLIC PT_GAMEPLUS=0)
endif()

target_include_directories(
    pt_engine
    PRIVATE "${CMAKE_CURRENT_BINARY_DIR}/generated"
)
add_dependencies(pt_engine pt_version)
target_include_directories(pt_engine PUBLIC src)
target_compile_definitions(
    pt_engine
    PUBLIC
        VK_NO_PROTOTYPES
        NOMINMAX
        WIN32_LEAN_AND_MEAN
        GLM_FORCE_DEPTH_ZERO_TO_ONE
        GLM_FORCE_RADIANS
)

target_link_libraries(pt_engine PUBLIC pt_thirdparty SDL3::SDL3-static)

if(WIN32)
    # dbghelp: the crash report (src/engine/core/crash_report.cpp) writes minidumps from pt_engine, so every test links it
    target_link_libraries(
        pt_engine
        PUBLIC gdi32 usp10 winhttp dbghelp ws2_32
    )
else()
    # Linux: the platform layer loads libcurl at run time (src/engine/platform/http.cpp); fonts are shaped with HarfBuzz
    target_link_libraries(pt_engine PUBLIC ${CMAKE_DL_LIBS} Threads::Threads)
endif()

if(WIN32)
    enable_language(RC)
    configure_file(assets/pt.rc.in "${CMAKE_CURRENT_BINARY_DIR}/pt.rc" @ONLY)
    # the icon is read by the resource compiler, which the build does not track: a new pt.ico left the old one in pt.exe
    set_source_files_properties(
        "${CMAKE_CURRENT_BINARY_DIR}/pt.rc"
        PROPERTIES OBJECT_DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/assets/pt.ico"
    )
endif()

# Sets shared properties for a target.
#
# On all compilers:
# - C++20 (without extensions)
#
# On GNU-like compilers:
# - -Wall
# - -Wpedantic
#
# On MSVC:
# - /W4
# - /utf-8
# - /EHsc
function(pt_setcommontargetproperties name)
    target_compile_features(${name} PRIVATE cxx_std_20)
    set_target_properties(${name} PROPERTIES CXX_EXTENSIONS OFF)

    if(CMAKE_CXX_COMPILER_FRONTEND_VARIANT STREQUAL "MSVC")
        target_compile_options(${name} PRIVATE /W4 /utf-8 /EHsc)
    elseif(CMAKE_CXX_COMPILER_FRONTEND_VARIANT STREQUAL "GNU")
        target_compile_options(${name} PRIVATE -Wall -Wpedantic)
    endif()
endfunction()

# Add an executable with common properties.
function(pt_add_executable name)
    add_executable(${name} ${ARGN})
    pt_setcommontargetproperties(${name})
endfunction()

# Add a library with common properties.
function(pt_add_library name)
    add_library(${name} ${ARGN})
    pt_setcommontargetproperties(${name})
endfunction()

# Add an executable with common properties, using the `pt_engine` target.
function(pt_add_engine_executable name)
    pt_add_executable(${name} ${ARGN})
    target_link_libraries(${name} PRIVATE pt_engine)
endfunction()

pt_add_engine_executable(pt WIN32 src/main.cpp)

if(WIN32)
    target_sources(pt PRIVATE "${CMAKE_CURRENT_BINARY_DIR}/pt.rc")
    target_link_libraries(pt PRIVATE dbghelp ole32 uuid)
endif()

# PT_BUILD_ID: the commit the executable is built from (plus "-dirty" for local changes), in the crash dump names and pt.log
add_custom_target(
    pt_build_id
    COMMAND
        "${CMAKE_COMMAND}"
        "-DOUT=${CMAKE_CURRENT_BINARY_DIR}/generated/build_id.h"
        "-DSOURCE=${CMAKE_CURRENT_SOURCE_DIR}" -P
        "${CMAKE_CURRENT_SOURCE_DIR}/cmake/build_id.cmake"
    BYPRODUCTS "${CMAKE_CURRENT_BINARY_DIR}/generated/build_id.h"
)

target_include_directories(pt PRIVATE "${CMAKE_CURRENT_BINARY_DIR}/generated")
add_dependencies(pt pt_build_id)

# the release 1.0 game: the loop browser stays locked until the game was finished once, its entries open as their loops are reached
# (PT_RELEASE_LOCKS, docs/gameplay.md loop browser); pt (developer) keeps every entry open. tools/ci/release.py
# builds it and tools/package.py --exe ships it as pt.exe
pt_add_engine_executable(pt_release WIN32 EXCLUDE_FROM_ALL src/main.cpp)

if(WIN32)
    target_sources(pt_release PRIVATE "${CMAKE_CURRENT_BINARY_DIR}/pt.rc")
    target_link_libraries(pt_release PRIVATE dbghelp ole32 uuid)
endif()

target_compile_definitions(pt_release PRIVATE PT_RELEASE_LOCKS=1)
target_include_directories(
    pt_release
    PRIVATE "${CMAKE_CURRENT_BINARY_DIR}/generated"
)

add_dependencies(pt_release pt_build_id)

pt_add_engine_executable(pt_gouge_input_test tests/gouge_input_test.cpp)
pt_add_engine_executable(pt_outro_input_test tests/outro_input_test.cpp)

set(PT_SETUP_PAYLOAD
    "${CMAKE_CURRENT_SOURCE_DIR}/dump/installer-20261002/native/payload.bin"
    CACHE FILEPATH
    "Installer payload"
)

# the setup's product strings (installer/Native/setup.rc.in); the setup checks its own integrity stamp (docs/installer.md)
set(PT_SETUP_PRODUCT "P.T. PC Port")
set(PT_SETUP_DESCRIPTION "P.T. PC Port Setup")

# the version resource: PT_VERSION's numbers (the environment's PT_VERSION, as cmake/version.cmake takes it, wins)
set(PT_SETUP_VERSION "${PT_VERSION}")
if(DEFINED ENV{PT_VERSION} AND NOT "$ENV{PT_VERSION}" STREQUAL "")
    set(PT_SETUP_VERSION "$ENV{PT_VERSION}")
endif()

string(REGEX REPLACE "-.*$" "" PT_SETUP_VERSION_NUMBERS "${PT_SETUP_VERSION}")
string(REPLACE "." ";" PT_SETUP_VERSION_LIST "${PT_SETUP_VERSION_NUMBERS}")

list(APPEND PT_SETUP_VERSION_LIST 0 0 0 0)
list(SUBLIST PT_SETUP_VERSION_LIST 0 4 PT_SETUP_VERSION_LIST)

string(REPLACE ";" "," PT_SETUP_VERSION_RC "${PT_SETUP_VERSION_LIST}")

if(WIN32 AND EXISTS "${PT_SETUP_PAYLOAD}")
    enable_language(RC)

    # the application manifest goes in as a resource (Common Controls 6 for the progress bar, docs/installer.md); the
    # linker must not make its own, or the two collide
    set(PT_SETUP_MANIFEST
        "${CMAKE_CURRENT_SOURCE_DIR}/installer/Native/setup.manifest"
    )

    configure_file(
        installer/Native/setup.rc.in
        "${CMAKE_CURRENT_BINARY_DIR}/private-setup.rc"
        @ONLY
    )

    set_source_files_properties(
        "${CMAKE_CURRENT_BINARY_DIR}/private-setup.rc"
        PROPERTIES
            OBJECT_DEPENDS
                "${PT_SETUP_PAYLOAD};${PT_SETUP_MANIFEST};${CMAKE_CURRENT_SOURCE_DIR}/assets/pt.ico"
    )

    pt_add_executable(
        pt_setup
        WIN32
        EXCLUDE_FROM_ALL
        installer/Native/setup.cpp
        src/engine/platform/http.cpp
        src/engine/platform/update_check.cpp
        "${CMAKE_CURRENT_BINARY_DIR}/private-setup.rc"
    )

    target_include_directories(
        pt_setup
        PRIVATE src installer/Native "${CMAKE_CURRENT_BINARY_DIR}/generated"
    )

    add_dependencies(pt_setup pt_version)
    target_compile_definitions(
        pt_setup
        PRIVATE NOMINMAX WIN32_LEAN_AND_MEAN UNICODE _UNICODE
    )

    target_link_libraries(
        pt_setup
        PRIVATE
            zlibstatic
            winhttp
            comctl32
            comdlg32
            shell32
            ole32
            uuid
            bcrypt
            gdiplus
    )

    target_link_options(pt_setup PRIVATE /MANIFEST:NO)

    if(MSVC)
        target_link_options(pt_setup PRIVATE /OPT:REF /OPT:ICF)
    endif()
endif()

if(NOT WIN32)
    # the Linux installer (installer/Native/setup_linux.cpp); tools/linux/attach_payload.py appends the payload after the build
    pt_add_executable(
        pt_setup_linux
        EXCLUDE_FROM_ALL
        installer/Native/setup_linux.cpp
        src/engine/platform/http.cpp
        src/engine/platform/update_check.cpp
        src/engine/platform/os.cpp
    )

    target_include_directories(
        pt_setup_linux
        PRIVATE src installer/Native "${CMAKE_CURRENT_BINARY_DIR}/generated"
    )

    add_dependencies(pt_setup_linux pt_version)
    target_link_libraries(
        pt_setup_linux
        PRIVATE zlibstatic ${CMAKE_DL_LIBS} Threads::Threads
    )
endif()

include(Upscalers)
include(EnhancedTextures)
include(OpenXR)
if(APPLE)
    include(macOS)
endif()

list(APPEND PT_UPSCALER_RUNTIME_FILES ${PT_OPENXR_RUNTIME_FILES})
list(APPEND PT_UPSCALER_NOTICES ${PT_OPENXR_NOTICES})

foreach(file ${PT_UPSCALER_RUNTIME_FILES})
    add_custom_command(
        TARGET pt
        POST_BUILD
        COMMAND
            ${CMAKE_COMMAND} -E copy_if_different "${file}"
            "$<TARGET_FILE_DIR:pt>"
        VERBATIM
    )
endforeach()

foreach(entry ${PT_UPSCALER_NOTICES})
    string(REPLACE "|" ";" parts "${entry}")
    list(GET parts 0 source)
    list(GET parts 1 name)
    add_custom_command(
        TARGET pt
        POST_BUILD
        COMMAND
            ${CMAKE_COMMAND} -E make_directory "$<TARGET_FILE_DIR:pt>/licenses"
        COMMAND
            ${CMAKE_COMMAND} -E copy_if_different "${source}"
            "$<TARGET_FILE_DIR:pt>/licenses/${name}"
        VERBATIM
    )
endforeach()

set(PT_SHADERS
    # verts
    shaders/fullscreen.vert
    shaders/mesh.vert
    shaders/shadow.vert
    shaders/ui_sprite.vert
    shaders/upscale_velocity.vert
    shaders/velocity.vert
    shaders/vfx_particle.vert
    shaders/volume.vert
    # frags
    shaders/banding.frag
    shaders/bright.frag
    shaders/compose.frag
    shaders/composite.frag
    shaders/debug.frag
    shaders/dof_blend.frag
    shaders/dof_blur.frag
    shaders/dof_down.frag
    shaders/dof_ratio.frag
    shaders/forward.frag
    shaders/fsblur.frag
    shaders/fxaa.frag
    shaders/gaussian.frag
    shaders/gbuffer.frag
    shaders/kawase.frag
    shaders/kawase_sum.frag
    shaders/light.frag
    shaders/light_contact.frag
    shaders/light_rt.frag
    shaders/mb_bake.frag
    shaders/mb_composite.frag
    shaders/mb_mcguire.frag
    shaders/mb_tile.frag
    shaders/mb_velocity.frag
    shaders/mirror_temporal.frag
    shaders/probe.frag
    shaders/probe_ao.frag
    shaders/probe_resolve.frag
    shaders/reflect_blend.frag
    shaders/reflect_blend_rt.frag
    shaders/reflect_layer.frag
    shaders/reflect_layer_rt.frag
    shaders/reflect_make.frag
    shaders/reflect_make_rt.frag
    shaders/reflect_temporal.frag
    shaders/screen_fx.frag
    shaders/shadow.frag
    shaders/ssao.frag
    shaders/ssao_blur.frag
    shaders/subliminal_string.frag
    shaders/subsurface.frag
    shaders/tonemap.frag
    shaders/ui_sprite.frag
    shaders/upscale_demod.frag
    shaders/upscale_motion.frag
    shaders/upscale_reactive.frag
    shaders/upscale_resolve.frag
    shaders/upscale_velocity.frag
    shaders/velocity.frag
    shaders/vfx_composite.frag
    shaders/vfx_depth_down.frag
    shaders/vfx_particle.frag
    shaders/xr_copy.frag
    # comps
    shaders/luminance.comp
    shaders/reflect_colour.comp
    shaders/rt_ao.comp
    shaders/rt_ao_filter.comp
    shaders/rt_skin.comp
)

set(PT_SPIRV_DIR "${CMAKE_CURRENT_BINARY_DIR}/shaders")
set(PT_SPIRV_OUTPUTS)
set(PT_GLSLC_DEFINES)

if(APPLE)
    # MoltenVK: no comparison sampler on the images[] array (shaders/lighting.glsl ShadowTap, docs/macos.md)
    list(APPEND PT_GLSLC_DEFINES -DPT_SHADOW_GATHER)
endif()

foreach(shader ${PT_SHADERS})
    get_filename_component(name ${shader} NAME)
    set(out ${PT_SPIRV_DIR}/${name}.spv)
    add_custom_command(
        OUTPUT ${out}
        COMMAND ${CMAKE_COMMAND} -E make_directory ${PT_SPIRV_DIR}
        COMMAND
            ${Vulkan_GLSLC_EXECUTABLE} --target-env=vulkan1.3 -O
            ${PT_GLSLC_DEFINES} -MD -MF ${out}.d ${shader} -o ${out}
        DEPENDS ${shader}
        DEPFILE ${out}.d
        VERBATIM
    )
    list(APPEND PT_SPIRV_OUTPUTS ${out})
endforeach()

add_custom_target(pt_shaders ALL DEPENDS ${PT_SPIRV_OUTPUTS})
add_dependencies(pt pt_shaders)

target_compile_definitions(
    pt_engine
    PUBLIC
        PT_SHADER_DIR="${PT_SPIRV_DIR}"
        PT_VOICE_MODEL_DIR="${CMAKE_CURRENT_BINARY_DIR}/voice"
)

target_include_directories(
    pt_engine
    PRIVATE ${PT_WHISPER_INCLUDE_DIRS} "${whisper_SOURCE_DIR}/ggml/src"
)

# whisper.cpp, ggml and its CPU variants in voice/ for every game target (tools/package.py ships them with any of these)
foreach(game pt pt_release)
    add_dependencies(${game} pt_voice_runtime)
endforeach()

pt_add_engine_executable(pt_tests tests/test_strcode.cpp)
pt_add_engine_executable(pt_voice_match_test tests/voice_match_test.cpp)
pt_add_engine_executable(pt_vfx_test tests/vfx_test.cpp)
pt_add_engine_executable(pt_collision_test tests/collision_test.cpp)
pt_add_engine_executable(pt_localization_test tests/localization_test.cpp)

pt_add_engine_executable(pt_texture_cache_test tests/texture_cache_test.cpp)
pt_add_engine_executable(pt_texture_job_test tests/texture_job_test.cpp)

set(PT_TEXTURE_DESCRIPTOR_SHADER
    "${CMAKE_CURRENT_BINARY_DIR}/tests/texture_descriptor.comp.spv"
)

add_custom_command(
    OUTPUT "${PT_TEXTURE_DESCRIPTOR_SHADER}"
    COMMAND
        ${CMAKE_COMMAND} -E make_directory "${CMAKE_CURRENT_BINARY_DIR}/tests"
    COMMAND
        ${Vulkan_GLSLC_EXECUTABLE} --target-env=vulkan1.3 -O
        "${CMAKE_CURRENT_SOURCE_DIR}/tests/texture_descriptor.comp" -o
        "${PT_TEXTURE_DESCRIPTOR_SHADER}"
    DEPENDS tests/texture_descriptor.comp
    VERBATIM
)

add_custom_target(
    pt_texture_descriptor_shader
    DEPENDS "${PT_TEXTURE_DESCRIPTOR_SHADER}"
)

pt_add_engine_executable(pt_texture_descriptor_test tests/texture_descriptor_test.cpp)
target_compile_definitions(
    pt_texture_descriptor_test
    PRIVATE PT_TEXTURE_DESCRIPTOR_SHADER="${PT_TEXTURE_DESCRIPTOR_SHADER}"
)

add_dependencies(pt_texture_descriptor_test pt_texture_descriptor_shader)

set(PT_REFLECTION_MIX_SHADER
    "${CMAKE_CURRENT_BINARY_DIR}/tests/reflection_mix_test.comp.spv"
)

add_custom_command(
    OUTPUT "${PT_REFLECTION_MIX_SHADER}"
    COMMAND
        ${Vulkan_GLSLC_EXECUTABLE} --target-env=vulkan1.3 -O -I
        "${CMAKE_CURRENT_SOURCE_DIR}/shaders"
        "${CMAKE_CURRENT_SOURCE_DIR}/tests/reflection_mix_test.comp" -o
        "${PT_REFLECTION_MIX_SHADER}"
    DEPENDS
        tests/reflection_mix_test.comp
        shaders/reflection_mix.glsl
        shaders/mirror_temporal.glsl
        shaders/flashlight_reactivity.glsl
    VERBATIM
)

add_custom_target(
    pt_reflection_mix_shader
    DEPENDS "${PT_REFLECTION_MIX_SHADER}"
)

pt_add_engine_executable(pt_reflection_mix_test tests/reflection_mix_test.cpp)
target_compile_definitions(
    pt_reflection_mix_test
    PRIVATE PT_REFLECTION_MIX_SHADER="${PT_REFLECTION_MIX_SHADER}"
)

add_dependencies(pt_reflection_mix_test pt_reflection_mix_shader)

pt_add_engine_executable(
    pt_dualsense_probe
    EXCLUDE_FROM_ALL
    tests/dualsense_hid_probe.cpp
)

pt_add_engine_executable(pt_audio_channel_layout_test tests/audio_channel_layout_test.cpp)
pt_add_engine_executable(pt_controller_feedback_test tests/controller_feedback_test.cpp)
pt_add_engine_executable(pt_photo_preset_test tests/photo_preset_test.cpp)
pt_add_engine_executable(pt_photo_panel_test tests/photo_panel_test.cpp)
pt_add_engine_executable(
    pt_controller_speaker_route_test
    tests/controller_speaker_route_test.cpp
)

pt_add_engine_executable(pt_hdr_output_test tests/hdr_output_test.cpp)
pt_add_engine_executable(pt_audio_test tests/audio_test.cpp)

pt_add_engine_executable(pt_graphics_preset_test tests/graphics_preset_test.cpp)

# the full screen blur's pulse start per game frame (0x9231D0), src/game/blur_sway.h
pt_add_engine_executable(pt_blur_sway_test tests/blur_sway_test.cpp)

pt_add_engine_executable(pt_unicode_paths_test tests/unicode_paths_test.cpp)
pt_add_engine_executable(pt_user_data_test tests/user_data_test.cpp)
pt_add_engine_executable(pt_settings_roundtrip_test tests/settings_roundtrip_test.cpp)

# the published body yaw that PlayerInputDir (0x915850) reads
pt_add_engine_executable(pt_player_publish_test tests/player_publish_test.cpp)

# the hair highlight's dominant light direction over time (0x8D12A0)
pt_add_engine_executable(pt_dominant_light_test tests/dominant_light_test.cpp)

# the VR mode's view mapping (src/engine/xr/xr_view.h, docs/vr.md)
pt_add_engine_executable(pt_vr_view_test tests/vr_view_test.cpp)

# the hardware check of FSR 3 frame generation (src/engine/render/upscale/upscale.h, docs/upscaling.md)
pt_add_engine_executable(pt_upscaler_gpu_test tests/upscaler_gpu_test.cpp)
pt_add_engine_executable(pt_peephole_look_test tests/peephole_look_test.cpp)

# the occluder stage of the original's light culling (0xD2A100, 0xD2AC10), src/engine/render/light_cull.cpp
pt_add_engine_executable(pt_light_cull_test tests/light_cull_test.cpp)

# the option defaults from the system language (0x91D7B0)
pt_add_engine_executable(pt_language_default_test tests/language_default_test.cpp)

# the boot load's outcomes and the failed writes behind the save dialogs (0x947C60, 0x9476D0)
pt_add_engine_executable(pt_save_status_test tests/save_status_test.cpp)

pt_add_engine_executable(pt_save_reset_test tests/save_reset_test.cpp)
pt_add_engine_executable(pt_multilingual_test tests/multilingual_test.cpp)

target_compile_definitions(
    pt_engine
    PUBLIC PT_FONT_DIR="${CMAKE_CURRENT_SOURCE_DIR}/assets/fonts"
)

add_custom_command(
    TARGET pt
    POST_BUILD
    COMMAND
        ${CMAKE_COMMAND} -E copy_directory
        "${CMAKE_CURRENT_SOURCE_DIR}/assets/fonts" "$<TARGET_FILE_DIR:pt>/fonts"
)

# the bundled fonts' SIL OFL notices also go to licenses/, beside the SDK notices
set(PT_FONT_LICENSES
    assets/fonts/OFL-NotoKufiArabic.txt
    assets/fonts/OFL-NotoNaskhArabic.txt
    assets/fonts/OFL-NotoSans.txt
    assets/fonts/OFL-NotoSansSC.txt
)

foreach(license ${PT_FONT_LICENSES})
    get_filename_component(license_name "${license}" NAME)
    add_custom_command(
        TARGET pt
        POST_BUILD
        COMMAND
            ${CMAKE_COMMAND} -E make_directory "$<TARGET_FILE_DIR:pt>/licenses"
        COMMAND
            ${CMAKE_COMMAND} -E copy_if_different "${license}"
            "$<TARGET_FILE_DIR:pt>/licenses/${license_name}"
        VERBATIM
    )
endforeach()

pt_add_engine_executable(pt_mouse_render_test tests/mouse_render_test.cpp)
pt_add_engine_executable(pt_skin_render_test tests/skin_render_test.cpp)

# the mod override index, mod.json and the mod script sandbox (docs/modding.md); no game files needed
# the platform layer (src/engine/platform: os, http, update check) on Windows and Linux, docs/linux.md
pt_add_engine_executable(pt_platform_test tests/platform_test.cpp)

pt_add_engine_executable(pt_mods_test tests/mods_test.cpp)
pt_add_engine_executable(pt_archive_model_camera_test tests/archive_model_camera_test.cpp)
pt_add_engine_executable(pt_display_modes_test tests/display_modes_test.cpp)
pt_add_engine_executable(pt_pipeline_cache_store_test tests/pipeline_cache_store_test.cpp)
