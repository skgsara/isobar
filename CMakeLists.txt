# Isobar — HF Weather-Fax (WEFAX) Decoder
# Single build system for macOS / Linux / Windows.
# See PORTING.md for the dependency story.
cmake_minimum_required(VERSION 3.16)

project(isobar
    VERSION 1.0.1
    LANGUAGES CXX)

# Expose the version to the source as ISOBAR_VERSION (used in the Details
# dialog). Generated so the version lives in one place — the project() call.
configure_file(
    ${CMAKE_SOURCE_DIR}/cmake/isobar_version.h.in
    ${CMAKE_BINARY_DIR}/isobar_version.h
    @ONLY)

# --- Build flags -------------------------------------------------------------
# Plain, readable C++17. gcc/clang use -std=c++17 -Wall -Wextra; MSVC uses its
# own flags. (Guarded up front rather than add-then-remove: remove_definitions
# can't undo add_compile_options reliably, so gcc flags leaked onto cl.exe.)
if(NOT CMAKE_BUILD_TYPE AND NOT CMAKE_CONFIGURATION_TYPES)
    set(CMAKE_BUILD_TYPE Release CACHE STRING "" FORCE)
endif()

if(MSVC)
    add_compile_options(/std:c++17 /W4 /permissive-)
    if(CMAKE_BUILD_TYPE STREQUAL "Release")
        add_compile_options(/O2)
    endif()
else()
    add_compile_options(
        -std=c++17
        -Wall
        -Wextra
        $<$<CONFIG:Release>:-O2>)
endif()

enable_testing()

# --- Dependency discovery ----------------------------------------------------
# FLTK has no single cross-platform entry point, so we try three in order and
# record which worked in ISOBAR_FLTK_MODE for the link step below:
#   1. fltk-config script  — Homebrew macOS, apt libfltk1.3-dev (Linux). Neither
#      ships a CMake config or .pc; fltk-config is the supported entry point.
#   2. find_package(CONFIG) — vcpkg on Windows (and any FLTKConfig.cmake install).
#   3. pkg-config           — systems with fltk.pc / fltk_images.pc.
find_program(FLTK_CONFIG_EXECUTABLE NAMES fltk-config)
set(ISOBAR_FLTK_MODE "")

# fltk-config is a Unix shell script emitting gcc-style flags (-lfltk, -L…),
# which don't work under MSVC. vcpkg ships it under tools/fltk/ but on Windows
# we must use the CMake config instead. Skip fltk-config entirely on WIN32.
if(FLTK_CONFIG_EXECUTABLE AND NOT WIN32)
    set(ISOBAR_FLTK_MODE "fltk-config")
    execute_process(COMMAND ${FLTK_CONFIG_EXECUTABLE} --cxxflags
        OUTPUT_VARIABLE FLTK_CXXFLAGS OUTPUT_STRIP_TRAILING_WHITESPACE)
    # --use-images pulls in libfltk_images (+libpng/libjpeg) — needed for
    # Fl_PNG_Image, used by the app icon and the Details dialog icon.
    execute_process(COMMAND ${FLTK_CONFIG_EXECUTABLE} --use-images --ldflags
        OUTPUT_VARIABLE FLTK_LDFLAGS  OUTPUT_STRIP_TRAILING_WHITESPACE)
    # fltk-config names -ljpeg/-lpng but only points -L at the FLTK and libpng
    # Cellar dirs, not where the other deps live. On Homebrew, prepend the
    # prefix lib dir so those resolve. (Linux distros put everything in the
    # default search path, so this is macOS-only.)
    if(APPLE)
        execute_process(COMMAND brew --prefix
            OUTPUT_VARIABLE HOMEBREW_PREFIX OUTPUT_STRIP_TRAILING_WHITESPACE
            RESULT_VARIABLE BREW_RC)
        if(BREW_RC EQUAL 0 AND EXISTS "${HOMEBREW_PREFIX}/lib")
            set(FLTK_LDFLAGS "-L${HOMEBREW_PREFIX}/lib ${FLTK_LDFLAGS}")
        endif()
    endif()
    # Keep each flag group intact: -weak_framework Foo must stay paired, and
    # SHELL: tells CMake not to de-duplicate or re-split the tokens.
    string(REPLACE " " ";" FLTK_CXXFLAGS "${FLTK_CXXFLAGS}")
else()
    find_package(FLTK CONFIG QUIET)
    # Record the FLTK target(s) to link. FLTK's target naming differs across
    # versions/sources: upstream 1.4 = fltk::fltk + fltk::images; upstream 1.3
    # config exports bare fltk/fltk_images; some vcpkg ports wrap in
    # unofficial::. Pick the first pair that exists.
    set(ISOBAR_FLTK_CORE "")
    set(ISOBAR_FLTK_IMAGES "")
    foreach(pair "fltk::fltk|fltk::images" "fltk|fltk_images" "unofficial::fltk::fltk|unofficial::fltk::images")
        string(REPLACE "|" ";" p ${pair})
        list(GET p 0 c)
        list(GET p 1 i)
        if(TARGET ${c})
            set(ISOBAR_FLTK_CORE ${c})
            if(TARGET ${i})
                set(ISOBAR_FLTK_IMAGES ${i})
            endif()
            break()
        endif()
    endforeach()
    if(ISOBAR_FLTK_CORE)
        set(ISOBAR_FLTK_MODE "config")
    elseif(FLTK_FOUND AND FLTK_LIBRARIES)
        # Legacy variable mode (1.3 module-style config). FLTK_IMAGES_LIBRARY
        # is singular in the module API; accept either name.
        set(ISOBAR_FLTK_MODE "config-vars")
        set(ISOBAR_FLTK_IMAGES_VAR ${FLTK_IMAGES_LIBRARIES} ${FLTK_IMAGES_LIBRARY})
    else()
        find_package(PkgConfig QUIET)
        if(PkgConfig_FOUND)
            # NOTE: pkg_check_modules(<prefix> IMPORTED_TARGET a b) makes ONE
            # imported target, PkgConfig::<prefix>, covering BOTH modules —
            # there is no PkgConfig::fltk_images. Link only PkgConfig::FLTK.
            pkg_check_modules(FLTK IMPORTED_TARGET fltk fltk_images)
            if(FLTK_FOUND)
                set(ISOBAR_FLTK_MODE "pkg-config")
            endif()
        endif()
    endif()
    if(NOT ISOBAR_FLTK_MODE)
        message(FATAL_ERROR
            "FLTK not found: install it (brew install fltk / apt install "
            "libfltk1.3-dev / vcpkg install fltk), put fltk-config on PATH, "
            "or pass -DCMAKE_TOOLCHAIN_FILE=<vcpkg>/scripts/buildsystems/vcpkg.cmake")
    endif()
endif()

# RtAudio: prefer a CMake config (vcpkg → RtAudio::rtaudio), then pkg-config
# (Homebrew/apt ship rtaudio.pc), then a manual find as a last resort. The
# manual find only recovers headers + the base lib, so config/pkg-config are
# preferred for pulling in the right per-OS audio link deps.
set(ISOBAR_RTAUDIO_MODE "")
find_package(RtAudio CONFIG QUIET)
if(RtAudio_FOUND)
    set(ISOBAR_RTAUDIO_MODE "config")
else()
    find_package(PkgConfig QUIET)
    if(PkgConfig_FOUND)
        pkg_check_modules(RTAUDIO IMPORTED_TARGET rtaudio)
        if(RTAUDIO_FOUND)
            set(ISOBAR_RTAUDIO_MODE "pkg-config")
        endif()
    endif()
endif()

# Wire an RtAudio link to a target via whichever discovery mode won. Used for
# isobar-gui and the audio dev tools (playwav, audio-restart-test) so the
# three stay in sync. The manual-find branch runs lazily so a missing RtAudio
# only fails targets that actually need audio, not the whole configure.
function(isobar_link_rtaudio target)
    if(ISOBAR_RTAUDIO_MODE STREQUAL "config")
        target_link_libraries(${target} PRIVATE RtAudio::rtaudio)
    elseif(ISOBAR_RTAUDIO_MODE STREQUAL "pkg-config")
        target_link_libraries(${target} PRIVATE PkgConfig::RTAUDIO)
    else()
        # Manual fallback so non-pkg-config/non-config RtAudio installs build.
        find_path(RTAUDIO_INCLUDE_DIR RtAudio.h)
        find_library(RTAUDIO_LIBRARY NAMES rtaudio)
        target_include_directories(${target} PRIVATE ${RTAUDIO_INCLUDE_DIR})
        target_link_libraries(${target} PRIVATE ${RTAUDIO_LIBRARY})
    endif()
endfunction()

# --- Core library (dependency-free C++17) ------------------------------------
add_library(isobar_core STATIC
    core/filters.cpp      core/filters.h
    core/decoder.cpp      core/decoder.h
    core/wavfile.cpp      core/wavfile.h
    core/syncscan.cpp     core/syncscan.h
    core/synfile.cpp      core/synfile.h
    core/fft.cpp          core/fft.h
    core/settings.cpp     core/settings.h
    core/tonedetect.cpp   core/tonedetect.h
    core/live.cpp         core/live.h
    core/resample.cpp     core/resample.h
    core/bmpfile.cpp      core/bmpfile.h
    core/palette.cpp      core/palette.h)
target_include_directories(isobar_core PUBLIC
    ${CMAKE_CURRENT_SOURCE_DIR}
    ${CMAKE_BINARY_DIR})   # for the generated isobar_version.h

# --- GUI sources (FLTK) ------------------------------------------------------
set(GUI_SRC
    gui/main.cpp
    gui/faxview.cpp       gui/faxview.h
    gui/scopeview.cpp     gui/scopeview.h
    gui/settingsdialog.cpp gui/settingsdialog.h
    gui/devicedialog.cpp  gui/devicedialog.h
    gui/colordialog.cpp   gui/colordialog.h
    gui/exportdialog.cpp  gui/exportdialog.h
    gui/autosavedialog.cpp gui/autosavedialog.h
    gui/noticedialog.cpp  gui/noticedialog.h
    gui/progressdialog.cpp gui/progressdialog.h
    gui/audio.cpp         gui/audio.h)

# --- isobar-decode: CLI decoder (core only) ----------------------------------
add_executable(isobar-decode cli/kgfax-decode.cpp)
target_link_libraries(isobar-decode PRIVATE isobar_core)

# --- isobar-gui: full GUI (core + FLTK + RtAudio) ----------------------------
add_executable(isobar-gui ${GUI_SRC})
target_link_libraries(isobar-gui PRIVATE isobar_core)
# FLTK link — mode picked during discovery above. fltk_images is pulled in
# (for Fl_PNG_Image) in every branch: by --use-images on fltk-config, by the
# separate fltk_images target in config mode, or inside PkgConfig::FLTK in
# pkg-config mode (a single target covering both modules).
if(ISOBAR_FLTK_MODE STREQUAL "fltk-config")
    target_compile_options(isobar-gui PRIVATE ${FLTK_CXXFLAGS})
    # On macOS the fltk-config link line has paired flags (-weak_framework Foo)
    # that whitespace-splitting would break, so pass it whole via SHELL: in a
    # link OPTION (ld64 is two-pass, so option ordering doesn't matter).
    # On Linux, GNU ld is single-pass: a link OPTION lands BEFORE the .o files,
    # leaving FLTK symbols unresolved. So split into a list there and feed it
    # through target_link_libraries, which CMake emits AFTER the objects.
    if(APPLE)
        target_link_options(isobar-gui PRIVATE "SHELL:${FLTK_LDFLAGS}")
    else()
        separate_arguments(FLTK_LDFLAGS_LIST UNIX_COMMAND "${FLTK_LDFLAGS}")
        target_link_libraries(isobar-gui PRIVATE ${FLTK_LDFLAGS_LIST})
    endif()
elseif(ISOBAR_FLTK_MODE STREQUAL "config")
    # ISOBAR_FLTK_CORE/IMAGES were resolved during discovery (probing the
    # fltk::fltk, bare-fltk, and unofficial::fltk shapes; vcpkg 1.3 exposes
    # the bare 'fltk' target). fltk_images brings Fl_PNG_Image + png/jpeg/z.
    target_link_libraries(isobar-gui PRIVATE ${ISOBAR_FLTK_CORE} ${ISOBAR_FLTK_IMAGES})
elseif(ISOBAR_FLTK_MODE STREQUAL "config-vars")
    # Legacy variable mode: a 1.3-style config that sets FLTK_LIBRARIES but no
    # targets. Fail loudly if empty — never reach the linker without FLTK.
    if(NOT FLTK_LIBRARIES)
        message(FATAL_ERROR
            "FLTK config found but exported no usable targets or libraries "
            "(FLTK_DIR=[${FLTK_DIR}]). Install a current FLTK (1.4 exports "
            "fltk::fltk) or ensure the config sets FLTK_LIBRARIES.")
    endif()
    target_include_directories(isobar-gui PRIVATE ${FLTK_INCLUDE_DIRS} ${FLTK_INCLUDE_DIR})
    target_link_libraries(isobar-gui PRIVATE ${FLTK_LIBRARIES} ${ISOBAR_FLTK_IMAGES_VAR})
elseif(ISOBAR_FLTK_MODE STREQUAL "pkg-config")
    target_link_libraries(isobar-gui PRIVATE PkgConfig::FLTK)
endif()
isobar_link_rtaudio(isobar-gui)

# Windows: FLTK's Win32 backends (Fl_Native_File_Chooser, Fl_Printer) need the
# common-control / GDI / OLE / common-dialog system libs. A raw-variable FLTK
# link doesn't pull these in via INTERFACE_LINK_LIBRARIES the way the imported
# targets do, so add them explicitly for the MSVC build.
if(WIN32)
    target_link_libraries(isobar-gui PRIVATE
        comctl32 ws2_32 gdi32 user32 shell32 ole32 oleaut32 uuid comdlg32)
endif()

# --- macOS .app bundle -------------------------------------------------------
# On macOS, isobar-gui becomes Isobar.app with the icon, Info.plist (carrying
# the mandatory NSMicrophoneUsageDescription), and an ad-hoc signature. On
# other platforms this block is skipped — the target stays a plain executable.
if(APPLE)
    # Bundle identity + version (also used inside Info.plist).
    set(MACOSX_BUNDLE_GUI_IDENTIFIER "com.sakuragawasara.isobar")
    set(MACOSX_BUNDLE_EXECUTABLE_NAME "isobar-gui")
    set(ISOBAR_VERSION "${PROJECT_VERSION}")
    configure_file(
        ${CMAKE_SOURCE_DIR}/macos/Info.plist.in
        ${CMAKE_BINARY_DIR}/Info.plist
        @ONLY)

    set_target_properties(isobar-gui PROPERTIES
        MACOSX_BUNDLE TRUE
        MACOSX_BUNDLE_INFO_PLIST ${CMAKE_BINARY_DIR}/Info.plist
        OUTPUT_NAME "Isobar")
    # Ship the .icns in Resources/ so Info.plist's CFBundleIconFile resolves,
    # and the 128px PNG so the runtime window-icon lookup (exe_dir()) finds it.
    set_source_files_properties(
        ${CMAKE_SOURCE_DIR}/assets/isobar.icns
        ${CMAKE_SOURCE_DIR}/assets/isobar-128.png
        PROPERTIES MACOSX_PACKAGE_LOCATION "Resources")
    target_sources(isobar-gui PRIVATE
        ${CMAKE_SOURCE_DIR}/assets/isobar.icns
        ${CMAKE_SOURCE_DIR}/assets/isobar-128.png)
endif()

# --- Install rules (drives CPack packaging) ----------------------------------
# Ship the two user-facing binaries. On macOS the GUI is a .app bundle; the
# BUNDLE destination puts the whole Isobar.app on the .dmg. On Linux/Windows
# the plain executables land in the package's bin/ (flat layout for a portable
# release — the runtime looks for kgfax.ini next to the exe either way).
install(TARGETS isobar-gui isobar-decode
        RUNTIME DESTINATION bin
        BUNDLE  DESTINATION .)
# The runtime window-icon lookup needs isobar-128.png next to the exe on the
# flat (non-bundle) layouts; ship it alongside the binaries there.
if(NOT APPLE)
    install(FILES assets/isobar-128.png DESTINATION bin)
endif()

# --- Tests & utilities -------------------------------------------------------
# The 7 below run headless and need only the source tree (some read
# "jmh sample.wav" via a relative path, so they run from the source root).
# playwav and audio-restart-test need real audio hardware → not in ctest.

# Self-contained core tests (no WAV, no deps).
add_executable(fft-test      cli/fft-test.cpp      core/fft.cpp)
add_executable(settings-test cli/settings-test.cpp)
target_link_libraries(settings-test PRIVATE isobar_core)
add_test(NAME fft-test      COMMAND fft-test)
add_test(NAME settings-test COMMAND settings-test)

# Tests that decode the JMH sample WAV (relative path → run from source root).
function(add_isobar_test name)
    add_executable(${name} cli/${name}.cpp)
    target_link_libraries(${name} PRIVATE isobar_core)
    add_test(NAME ${name} COMMAND ${name})
    set_tests_properties(${name} PROPERTIES WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})
endfunction()

add_isobar_test(rpm60-test)
add_isobar_test(fallback-test)
add_isobar_test(tone-test)
add_isobar_test(live-test)

# resample-test needs filters.cpp directly in addition to core.
add_executable(resample-test cli/resample-test.cpp)
target_link_libraries(resample-test PRIVATE isobar_core)
add_test(NAME resample-test COMMAND resample-test)
set_tests_properties(resample-test PROPERTIES WORKING_DIRECTORY ${CMAKE_SOURCE_DIR})

# Dev utilities that need real audio hardware (not run by ctest).
add_executable(playwav cli/playwav.cpp core/wavfile.cpp core/filters.cpp core/resample.cpp)
isobar_link_rtaudio(playwav)

add_executable(audio-restart-test cli/audio-restart-test.cpp gui/audio.cpp core/resample.cpp core/filters.cpp)
isobar_link_rtaudio(audio-restart-test)

# --- Packaging (CPack) -------------------------------------------------------
# Per-OS release formats (decision 2026-07-29): macOS = DragNDrop (.dmg),
# Windows = ZIP (.zip), Linux = TBZ2 (portable tarball). AppImage is left out
# for now — it needs linuxdeploy/appimagetool at package time, which is extra
# machinery for a hobbyist release; the tarball + the distro's own FLTK/RtAudio
# is the simple Linux story. Run `cpack` after a build to produce packages.
set(CPACK_PACKAGE_NAME "Isobar")
set(CPACK_PACKAGE_VENDOR "Sara Sakuragawa")
set(CPACK_PACKAGE_VERSION "${PROJECT_VERSION}")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY
    "Cross-platform HF weather-fax (WEFAX) decoder — interoperable with KG-FAX")
set(CPACK_RESOURCE_FILE_LICENSE "${CMAKE_SOURCE_DIR}/LICENSE")
set(CPACK_RESOURCE_FILE_README "${CMAKE_SOURCE_DIR}/README.md")
set(CPACK_PACKAGE_HOMEPAGE_URL "https://github.com/sakuragawasara/isobar")
if(APPLE)
    set(CPACK_DMG_FORMAT "UDBZ")
    set(CPACK_DMG_VOLUME_NAME "Isobar")
    set(CPACK_BUNDLE_NAME "Isobar")
endif()
# One native generator per OS (CPack produces that package from the install tree).
set(CPACK_GENERATOR "")
if(APPLE)
    list(APPEND CPACK_GENERATOR "DragNDrop")
elseif(WIN32)
    list(APPEND CPACK_GENERATOR "ZIP")
else()
    list(APPEND CPACK_GENERATOR "TBZ2")
endif()
include(CPack)

