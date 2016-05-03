# ---[ Create project
project(ReverseTale-Headless)

# ---[ Parse all prefixes
include(cmake-common/Utils.cmake)
String2List(${CMAKE_PREFIX_PATH} CMAKE_PREFIX_PATH)
String2List(${CMAKE_MODULE_PATH} CMAKE_MODULE_PATH)

# ---[ Find all source and header files
file(GLOB_RECURSE sources_Headless      src/*.cpp     src/*.h)
set(Headless_SOURCES
    ${sources_Headless}
)

# ---[ Load dependencies
if (${ReverseTale_DEV})
    include_directories(${CMAKE_SOURCE_DIR}/src/Library/include)
else()
    find_package(ReverseTale-Library   REQUIRED)
endif()

find_package(inih REQUIRED)

# ---[ Add definitions and DLL
if (NOT ${DEBUG})
	add_definitions(-DNDEBUG)
endif()

add_executable(ReverseTale-Headless ${Headless_SOURCES})

# ---[ Link dependencies libraries
target_link_libraries(ReverseTale-Headless
    PUBLIC ReverseTale-Library
    PUBLIC "$<TARGET_FILE:inih>"
)

get_target_property(INIH_INCLUDE_DIR inih INTERFACE_INCLUDE_DIRECTORIES)
target_include_directories(ReverseTale-Headless
  PUBLIC ${INIH_INCLUDE_DIR}
  INTERFACE "$<INSTALL_INTERFACE:include>"
)

# ---[ Set C++11 standard
set_property(TARGET ReverseTale-Headless PROPERTY CXX_STANDARD 11)
target_compile_features(ReverseTale-Headless PRIVATE cxx_decltype cxx_nullptr cxx_variadic_templates)

set_target_properties(ReverseTale-Headless PROPERTIES
    OUTPUT_NAME Headless
    RUNTIME_OUTPUT_DIRECTORY ${PROJECT_SOURCE_DIR}/bin
    RUNTIME_OUTPUT_DIRECTORY_DEBUG ${PROJECT_SOURCE_DIR}/bin
    RUNTIME_OUTPUT_DIRECTORY_RELEASE ${PROJECT_SOURCE_DIR}/bin)

include(CMakePackageConfigHelpers)
set(config_install_dir lib/cmake/ReverseTale-Headless)
set(version_config ${PROJECT_BINARY_DIR}/ReverseTale-Headless-config-version.cmake)
set(project_config ${PROJECT_BINARY_DIR}/ReverseTale-Headless-config.cmake)
set(targets_export_name ReverseTale-Headless-targets)

# Generate the version, config and target files into the build directory.
write_basic_package_version_file(
  ${version_config}
  VERSION 0.0.1
  COMPATIBILITY AnyNewerVersion)
configure_package_config_file(
  ${PROJECT_SOURCE_DIR}/config.cmake.in
  ${project_config}
  INSTALL_DESTINATION ${config_install_dir})
export(TARGETS ReverseTale-Headless FILE ${PROJECT_BINARY_DIR}/${targets_export_name}.cmake)

install(TARGETS ReverseTale-Headless
	EXPORT ${targets_export_name}
	RUNTIME DESTINATION bin/
  	LIBRARY DESTINATION lib/
  	ARCHIVE DESTINATION lib/)

install(FILES ${project_config} ${version_config} DESTINATION ${config_install_dir})
install(EXPORT ${targets_export_name} DESTINATION ${config_install_dir})
