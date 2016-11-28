cmake_minimum_required (VERSION 3.0)

project(OpenCL VERSION 2.1)

set (CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set (CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

set (OPENCL_ICD_LOADER_SOURCES icd.c icd_dispatch.c)

if ("${CMAKE_SYSTEM_NAME}" STREQUAL "Linux")
    list (APPEND OPENCL_ICD_LOADER_SOURCES icd_linux.c icd_exports.map)
else ()
    list (APPEND OPENCL_ICD_LOADER_SOURCES icd_windows.c OpenCL.def)
    include_directories ($ENV{DXSDK_DIR}/Include)
endif ()

if(APPLE) #for some reason on apple opencl is expected under OpenCL. And oddly enough, still need the include/CL version for ICD to build
    set(opencl_include_dir "${CMAKE_CURRENT_BINARY_DIR}/include")
    if(EXISTS "${CMAKE_CURRENT_LIST_DIR}/include/CL")
        file(COPY "${CMAKE_CURRENT_LIST_DIR}/include/CL/" DESTINATION "${opencl_include_dir}/OpenCL")
    endif()
    include_directories("${opencl_include_dir}")
else()
   set(opencl_include_dir "${CMAKE_CURRENT_LIST_DIR}/include")
endif()
include_directories("${CMAKE_CURRENT_LIST_DIR}/include")


add_library (OpenCL SHARED ${OPENCL_ICD_LOADER_SOURCES})
set_target_properties (OpenCL PROPERTIES VERSION "2.1" SOVERSION "1")

if ("${CMAKE_SYSTEM_NAME}" STREQUAL "Linux")
    set_target_properties (OpenCL PROPERTIES LINK_FLAGS "-pthread -Wl,--version-script -Wl,${CMAKE_SOURCE_DIR}/icd_exports.map")
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

configure_file("Config.cmake.in" "${project_config}" @ONLY)

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
    FILES "${project_config}" "${version_config}"
    DESTINATION "${config_install_dir}"
)

install(
    EXPORT "${targets_export_name}"
    NAMESPACE "${namespace}"
    DESTINATION "${config_install_dir}"
)