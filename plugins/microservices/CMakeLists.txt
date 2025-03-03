cmake_minimum_required(VERSION 3.12.0 FATAL_ERROR)
# target_link_libraries for object targets

# create phony target to build everything in plugins/microservices folder
add_custom_target(all-plugins-microservices)
add_dependencies(all-plugins_no_database all-plugins-microservices)

set(
  IRODS_MICROSERVICE_PLUGINS
  sync_to_archive
  update_unixfilesystem_resource_free_space
  atomic_apply_acl_operations
  atomic_apply_metadata_operations
  get_agent_pid
  touch
  get_open_data_obj_l1desc_index
  get_file_descriptor_info
  get_hostname
)

foreach(plugin IN LISTS IRODS_MICROSERVICE_PLUGINS)
  string(TOUPPER ${plugin} PLUGIN_UPPERCASE)
  set(plugin_target "irods_microservice_plugin-${plugin}")
  set(plugin_name "msi_${plugin}")

  add_library(
    ${plugin_target}
    MODULE
    "${CMAKE_CURRENT_SOURCE_DIR}/src/${plugin}.cpp"
  )
  set_property(TARGET ${plugin_target} PROPERTY LIBRARY_OUTPUT_NAME ${plugin_name})
  target_link_libraries(
    ${plugin_target}
    PRIVATE
    irods_common
    irods_server
    "${IRODS_EXTERNALS_FULLPATH_BOOST}/lib/libboost_filesystem.so"
    "${IRODS_EXTERNALS_FULLPATH_BOOST}/lib/libboost_system.so"
    "${IRODS_EXTERNALS_FULLPATH_FMT}/lib/libfmt.so"
  )
  target_include_directories(
    ${plugin_target}
    PRIVATE
    "${IRODS_EXTERNALS_FULLPATH_BOOST}/include"
    "${IRODS_EXTERNALS_FULLPATH_FMT}/include"
    "${IRODS_EXTERNALS_FULLPATH_SPDLOG}/include"
  )
  target_compile_definitions(
    ${plugin_target}
    PRIVATE
    ${IRODS_COMPILE_DEFINITIONS_PRIVATE}
    ENABLE_RE
    IRODS_ENABLE_SYSLOG
  )

  add_dependencies(all-plugins-microservices ${plugin_target})
  install(
    TARGETS
    ${plugin_target}
    LIBRARY
    DESTINATION "${IRODS_PLUGINS_DIRECTORY}/microservices"
    COMPONENT ${IRODS_PACKAGE_COMPONENT_SERVER_NAME}
  )
endforeach()

set_property(TARGET irods_microservice_plugin-sync_to_archive PROPERTY LIBRARY_OUTPUT_NAME msisync_to_archive)
