cmake_minimum_required( VERSION 3.6 )

set( project_name png )
set( PNG_VERSION 1.6.29 )

project( ${project_name} VERSION ${PNG_VERSION} )

if( MSVC )
    add_definitions(/MP)
endif()

set( source_dir ${CMAKE_CURRENT_SOURCE_DIR}/jni )

set( headers
    ${source_dir}/pnginfo.h
    ${source_dir}/config.h
    ${source_dir}/png.h
    ${source_dir}/pngconf.h
    ${source_dir}/pngdebug.h
    ${source_dir}/pnglibconf.h
    ${source_dir}/pngprefix.h
    ${source_dir}/pngpriv.h
    ${source_dir}/pngstruct.h
)

set( sources
    ${source_dir}/arm/arm_init.c
    ${source_dir}/arm/filter_neon.S
    ${source_dir}/arm/filter_neon_intrinsics.c
    ${source_dir}/png.c
    ${source_dir}/pngerror.c
    ${source_dir}/pngget.c
    ${source_dir}/pngmem.c
    ${source_dir}/pngpread.c
    ${source_dir}/pngread.c
    ${source_dir}/pngrio.c
    ${source_dir}/pngrtran.c
    ${source_dir}/pngrutil.c
    ${source_dir}/pngset.c
    ${source_dir}/pngtest.c
    ${source_dir}/pngtrans.c
    ${source_dir}/pngwio.c
    ${source_dir}/pngwrite.c
    ${source_dir}/pngwtran.c
    ${source_dir}/pngwutil.c
)

# These variables *may* be used in png-config.cmake.in
set( zlib_import_target zlib::zlib )
set( zlib_find_package_type CONFIG )
set( zlib_find_package_name zlib )

# Prioritize zlib config package.
# Provided by zlib on github: https://github.com/madler/zlib
find_package( zlib CONFIG )

if( NOT zlib_DIR )
    # As a last resort, use the module package
    find_package( ZLIB MODULE REQUIRED )

    # These variables *may* be used in png-config.cmake.in
    set( zlib_import_target ZLIB::ZLIB )
    set( zlib_find_package_type MODULE )
    set( zlib_find_package_name ZLIB )
endif()

if( WIN32 )
    set( CMAKE_DEBUG_POSTFIX -debug )
endif()

# If you want a shared library instead of static, specify the BUILD_SHARED_LIBS
# variable (which is a built-in CMake option).
add_library( ${project_name} ${sources} ${headers} )

set_target_properties( ${project_name} PROPERTIES OUTPUT_NAME ${project_name}-${PNG_VERSION} )
target_link_libraries( ${project_name} PUBLIC ${zlib_import_target} )
target_include_directories( ${project_name} PUBLIC
    $<BUILD_INTERFACE:${source_dir}>
    $<INSTALL_INTERFACE:include>
)

############################################################
## INSTALL
############################################################

include( CMakePackageConfigHelpers )

set( PNG_EXPORT_DIR     lib/png-${PNG_VERSION}/cmake )
set( PNG_EXPORT_NAME    png-targets )

install( TARGETS png EXPORT ${PNG_EXPORT_NAME}
    RUNTIME DESTINATION bin
    LIBRARY DESTINATION lib
    ARCHIVE DESTINATION lib
    INCLUDES DESTINATION include
)

install( FILES ${headers} DESTINATION include )

install(
    EXPORT ${PNG_EXPORT_NAME}
    DESTINATION ${PNG_EXPORT_DIR}
    NAMESPACE png::
)

configure_file( cmake/png-config.cmake.in png-config.cmake @ONLY )

write_basic_package_version_file(
    png-config-version.cmake
    VERSION ${PNG_VERSION}
    COMPATIBILITY AnyNewerVersion
)

install(
    FILES
        ${CMAKE_CURRENT_BINARY_DIR}/png-config.cmake
        ${CMAKE_CURRENT_BINARY_DIR}/png-config-version.cmake
    DESTINATION ${PNG_EXPORT_DIR}
)
