cmake_minimum_required (VERSION 3.1)

project (OPENCL_ICD_LOADER VERSION 2.1)

option(BUILD_SHARED_LIBS "Build shared libs" ON)
option(OPENCL_TESTS "Enable building tests" ON)

set (CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set (CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)

set (OPENCL_ICD_LOADER_SOURCES icd.c icd_dispatch.c)

if (WIN32)
    list (APPEND OPENCL_ICD_LOADER_SOURCES icd_windows.c icd_windows_hkr.c OpenCL.def OpenCL.rc)
    include_directories ($ENV{DXSDK_DIR}/Include)
else ()
    list (APPEND OPENCL_ICD_LOADER_SOURCES icd_linux.c icd_exports.map)
endif ()

# Change this to point to a directory containing OpenCL header directory "CL"
# OR copy OpenCL headers to ./inc/CL/
if (NOT DEFINED OPENCL_INCLUDE_DIRS)
    set (OPENCL_INCLUDE_DIRS ./inc)
endif ()

include_directories (${OPENCL_INCLUDE_DIRS})

add_library (OpenCL SHARED ${OPENCL_ICD_LOADER_SOURCES})
set_target_properties (OpenCL PROPERTIES VERSION "2.1" SOVERSION "1")

if (WIN32)
    target_link_libraries (OpenCL cfgmgr32.lib)
else()
    if (APPLE)
        set_target_properties (OpenCL PROPERTIES LINK_FLAGS "-pthread")
    else ()
        set_target_properties (OpenCL PROPERTIES LINK_FLAGS "-pthread -Wl,--version-script -Wl,${CMAKE_CURRENT_SOURCE_DIR}/icd_exports.map")
    endif ()
endif ()

target_link_libraries (OpenCL ${CMAKE_DL_LIBS})

#Installation
set(generated_dir "${CMAKE_CURRENT_BINARY_DIR}/generated")
set(config_install_dir "lib/cmake/${PROJECT_NAME}")
set(include_install_dir "include")

set(version_config "${generated_dir}/${PROJECT_NAME}ConfigVersion.cmake")
set(project_config "${generated_dir}/${PROJECT_NAME}Config.cmake")
set(targets_export_name "${PROJECT_NAME}Targets")
set(namespace "${PROJECT_NAME}::")

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    "${version_config}" COMPATIBILITY SameMajorVersion
)

install(
    TARGETS OpenCL
    EXPORT "${targets_export_name}"
    LIBRARY DESTINATION "lib"
    ARCHIVE DESTINATION "lib"
    RUNTIME DESTINATION "bin"
    INCLUDES DESTINATION "${include_install_dir}"
)

install(
    DIRECTORY "${opencl_include_dir}/"
    DESTINATION "${include_install_dir}"
)

install(
    EXPORT "${targets_export_name}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)

if (OPENCL_TESTS)
	enable_testing()
	add_subdirectory (test)
endif()
