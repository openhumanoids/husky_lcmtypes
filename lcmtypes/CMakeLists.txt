if(WIN32)
  set(CONFIG_INSTALL_DIR CMake)
else()
  set(CONFIG_INSTALL_DIR lib${LIB_SUFFIX}/${PROJECT_NAME}/cmake)
endif()

if(PYTHONINTERP_FOUND)
  set(python_args PYTHON_SOURCES python_install_sources)
endif()
if(JAVA_FOUND AND TARGET lcm-java)
  set(java_args JAVA_SOURCES java_sources)
endif()

lcm_wrap_types(
  C_EXPORT husky_lcmtypes
  C_SOURCES c_sources
  C_HEADERS c_install_headers
  CPP_HEADERS cpp_install_headers
  CREATE_C_AGGREGATE_HEADER
  CREATE_CPP_AGGREGATE_HEADER
  ${python_args}
  ${java_args}
  husky_status_t.lcm
  )


lcm_add_library(husky-lcmtypes C ${c_sources} ${c_install_headers})
generate_export_header(husky-lcmtypes
  BASE_NAME husky_lcmtypes)
target_include_directories(husky-lcmtypes INTERFACE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>)

lcm_add_library(husky-lcmtypes-cpp CPP ${cpp_install_headers})
target_include_directories(husky-lcmtypes-cpp INTERFACE
  $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}>)

lcm_install_headers(
  ${CMAKE_CURRENT_BINARY_DIR}/husky_lcmtypes_export.h
  ${c_install_headers}
  ${cpp_install_headers}
  DESTINATION include/lcmtypes
)

install(TARGETS husky-lcmtypes husky-lcmtypes-cpp
  EXPORT ${PROJECT_NAME}Targets
  RUNTIME DESTINATION bin
  LIBRARY DESTINATION lib${LIB_SUFFIX}
  ARCHIVE DESTINATION lib${LIB_SUFFIX}
  INCLUDES DESTINATION include/lcmtypes
)

if(PYTHONINTERP_FOUND)
  lcm_install_python(${python_install_sources})
endif()

if(JAVA_FOUND)
  if(NOT TARGET lcm-java)
    message(STATUS "lcm-java not found: not building Java LCM type bindings")
  else()
    include(UseJava)
    add_jar(husky-lcmtypes-jar
      OUTPUT_NAME husky-lcmtypes
      INCLUDE_JARS lcm-java
      SOURCES ${java_sources}
    )

    install_jar(husky-lcmtypes-jar share/java)

    set(HUSKY-LCMTYPES_INCLUDE_JAVA
      "include(\${CMAKE_CURRENT_LIST_DIR}/${PROJECT_NAME}JavaTargets.cmake)"
    )
    export_jars(
      FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}JavaTargets.cmake
      TARGETS husky-lcmtypes-jar
    )
    install_jar_exports(
      FILE ${PROJECT_NAME}JavaTargets.cmake
      DESTINATION ${CONFIG_INSTALL_DIR}
      TARGETS husky-lcmtypes-jar
    )
  endif()
endif()

configure_package_config_file(
  ${PROJECT_SOURCE_DIR}/${PROJECT_NAME}Config.cmake.in
  ${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
  INSTALL_DESTINATION ${CONFIG_INSTALL_DIR}
)

# Exported targets for build directory
export(
  TARGETS husky-lcmtypes husky-lcmtypes-cpp
  FILE ${PROJECT_BINARY_DIR}/${PROJECT_NAME}Targets.cmake
)

# Exported targets for installation
install(EXPORT ${PROJECT_NAME}Targets
  DESTINATION ${CONFIG_INSTALL_DIR}
  FILE ${PROJECT_NAME}Targets.cmake
)

# Install package configuration files
install(FILES
  ${PROJECT_BINARY_DIR}/${PROJECT_NAME}Config.cmake
  DESTINATION ${CONFIG_INSTALL_DIR}
)
