cmake_minimum_required(VERSION 3.21)
project(opabinia-desktop VERSION 1.0.0 LANGUAGES CXX OBJCXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_OBJCXX_STANDARD 17)
set(CMAKE_OSX_DEPLOYMENT_TARGET "13.0")
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

if(NOT CMAKE_CONFIGURATION_TYPES AND NOT CMAKE_BUILD_TYPE)
  set(CMAKE_BUILD_TYPE Release)
endif()

if(NOT CMAKE_CXX_COMPILER_ID MATCHES "Clang")
  message(WARNING "opabinia-desktop is developed for LLVM/Clang. Current: ${CMAKE_CXX_COMPILER_ID}")
endif()

add_compile_options(-fobjc-arc -Wall -Wextra -Wno-unused-parameter)

# Engine is extras/opalib (git submodule from git@github.com:DHANUSH-web/opalib.git, main).
set(DCMM_BUILD_TESTS OFF CACHE BOOL "Disable engine tests when nested" FORCE)
set(DCMM_BUILD_CLI OFF CACHE BOOL "Disable engine CLI when nested" FORCE)
if(EXISTS "${CMAKE_CURRENT_SOURCE_DIR}/extras/opalib/CMakeLists.txt")
  add_subdirectory(extras/opalib)
else()
  message(FATAL_ERROR "opalib not found. Run: make init  (clones extras/opalib via SSH, GitHub main)")
endif()

set(UI_SOURCES
  src/main.mm
  src/AppDelegate.mm
  src/SystemInfo.cpp
  src/ui/Widgets.mm
  src/ui/SidebarView.mm
  src/ui/DashboardView.mm
  src/ui/ResultsView.mm
  src/ui/DuplicatesView.mm
  src/ui/LargeFilesView.mm
  src/ui/UninstallerView.mm
  src/ui/SpaceLensView.mm
  src/ui/DevCornerView.mm
  src/ui/MaintenanceView.mm
  src/ui/SettingsView.mm
  src/ui/MainWindowController.mm
)

add_executable(Opabinia MACOSX_BUNDLE ${UI_SOURCES})
target_include_directories(Opabinia PRIVATE include)
target_link_libraries(Opabinia PRIVATE
  dcmm
  "-framework Cocoa"
  "-framework AppKit"
  "-framework Foundation"
  "-framework QuartzCore"
  "-framework Quartz"
  "-framework Collaboration"
  "-framework IOKit"
  "-framework DiskArbitration"
  "-framework SystemConfiguration"
  "-framework CoreFoundation"
  "-framework LocalAuthentication"
  "-weak_framework Symbols"
)

set(ICON_SOURCE "${CMAKE_SOURCE_DIR}/resources/opabinia-app-icon.icon")
set(ICON_GEN_DIR "${CMAKE_BINARY_DIR}/generated/AppIcon")
set(ICON_CAR "${ICON_GEN_DIR}/Assets.car")
set(ICON_ICNS "${ICON_GEN_DIR}/opabinia-app-icon.icns")
add_custom_command(
  OUTPUT "${ICON_CAR}" "${ICON_ICNS}"
  COMMAND "${CMAKE_COMMAND}" -E make_directory "${ICON_GEN_DIR}"
  COMMAND xcrun actool "${ICON_SOURCE}"
          --compile "${ICON_GEN_DIR}"
          --output-format human-readable-text
          --notices
          --warnings
          --app-icon opabinia-app-icon
          --include-all-app-icons
          --platform macosx
          --minimum-deployment-target 26.0
          --output-partial-info-plist "${ICON_GEN_DIR}/partial.plist"
  DEPENDS
    "${ICON_SOURCE}/icon.json"
    "${ICON_SOURCE}/Assets/Opa.svg"
  VERBATIM
)
target_sources(Opabinia PRIVATE "${ICON_CAR}" "${ICON_ICNS}")
set_source_files_properties("${ICON_CAR}" "${ICON_ICNS}" PROPERTIES
  MACOSX_PACKAGE_LOCATION Resources
  GENERATED TRUE
)

set_target_properties(Opabinia PROPERTIES
  MACOSX_BUNDLE TRUE
  MACOSX_BUNDLE_INFO_PLIST "${CMAKE_SOURCE_DIR}/resources/Info.plist.in"
  MACOSX_BUNDLE_BUNDLE_NAME "Opabinia"
  MACOSX_BUNDLE_BUNDLE_VERSION "${PROJECT_VERSION}"
  MACOSX_BUNDLE_SHORT_VERSION_STRING "${PROJECT_VERSION}"
  MACOSX_BUNDLE_GUI_IDENTIFIER "com.deepclean.Opabinia"
  MACOSX_BUNDLE_ICON_FILE "opabinia-app-icon"
  OUTPUT_NAME "Opabinia"
)

install(TARGETS Opabinia BUNDLE DESTINATION . COMPONENT Runtime)

option(DCMM_DESKTOP_BUILD_TESTS "Build GoogleTest suite for the desktop app" ON)
if(DCMM_DESKTOP_BUILD_TESTS)
  include(FetchContent)
  FetchContent_Declare(
    googletest
    URL https://github.com/google/googletest/archive/refs/tags/v1.15.2.zip
    DOWNLOAD_EXTRACT_TIMESTAMP TRUE
  )
  set(gtest_force_shared_crt ON CACHE BOOL "" FORCE)
  set(INSTALL_GTEST OFF CACHE BOOL "" FORCE)
  FetchContent_MakeAvailable(googletest)
  enable_testing()
  add_executable(opabinia-tests
    src/SystemInfo.cpp
    tests/test_engine_link.cpp
    tests/test_modules.cpp
    tests/test_navigation.cpp
    tests/test_mymac.cpp
    tests/test_smart_scan.cpp
    tests/test_deep_scan.cpp
    tests/test_privacy.cpp
    tests/test_large_files.cpp
    tests/test_duplicates.cpp
    tests/test_space_lens.cpp
    tests/test_uninstaller.cpp
    tests/test_maintenance.cpp
    tests/test_settings.cpp
  )
  target_include_directories(opabinia-tests PRIVATE include tests)
  target_link_libraries(opabinia-tests PRIVATE
    dcmm
    GTest::gtest_main
    "-framework IOKit"
    "-framework DiskArbitration"
    "-framework SystemConfiguration"
    "-framework CoreFoundation"
  )
  include(GoogleTest)
  gtest_discover_tests(opabinia-tests DISCOVERY_MODE PRE_TEST)
endif()

set(CPACK_GENERATOR "DragNDrop")
set(CPACK_PACKAGE_NAME "Opabinia")
set(CPACK_PACKAGE_VENDOR "Opabinia")
set(CPACK_PACKAGE_VERSION "${PROJECT_VERSION}")
set(CPACK_PACKAGE_FILE_NAME "Opabinia-${PROJECT_VERSION}")
set(CPACK_PACKAGE_DESCRIPTION_SUMMARY "Clean junk from your Mac")
set(CPACK_DMG_VOLUME_NAME "Opabinia")
set(CPACK_DMG_FORMAT "UDZO")
set(CPACK_PACKAGE_ICON "${CMAKE_SOURCE_DIR}/resources/AppIcon.icns")
set(CPACK_DMG_BACKGROUND_IMAGE "${CMAKE_SOURCE_DIR}/resources/dmg-background.png")
set(CPACK_DMG_DS_STORE_SETUP_SCRIPT "${CMAKE_SOURCE_DIR}/resources/dmg-setup.applescript")
set(CPACK_COMMAND_HDIUTIL "${CMAKE_SOURCE_DIR}/resources/hdiutil-cpack.sh")
set(CPACK_COMPONENTS_ALL Runtime)
set(CPACK_COMPONENT_RUNTIME_REQUIRED TRUE)
set(CPACK_INSTALL_CMAKE_PROJECTS "${CMAKE_BINARY_DIR};${PROJECT_NAME};Runtime;/")
include(CPack)
