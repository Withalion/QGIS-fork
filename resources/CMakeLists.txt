include(CopyResources)

# Folders to install
file(GLOB_RECURSE CPT_CITY_QGIS_MIN_FILES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} cpt-city-qgis-min/*)
file(GLOB_RECURSE THEMES_FILES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} themes/*)
file(GLOB_RECURSE DATA_FILES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} data/*)
file(GLOB_RECURSE METADATA_ISO_FILES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} metadata-ISO/*)
file(GLOB_RECURSE PALETTES_FILES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} palettes/*)

# Individual files to install (for srs.db, see below)
set(RESOURCES_FILES
  qgis.db
  symbology-style.xml
  spatialite.db
  customization.xml
  2to3migration.txt
  qgis_global_settings.ini
  ${CPT_CITY_QGIS_MIN_FILES}
  ${THEMES_FILES}
  ${DATA_FILES}
  ${METADATA_ISO_FILES}
  ${PALETTES_FILES}
)

# OpenCL, also install opencl_programs
if (HAVE_OPENCL)
  file(GLOB_RECURSE OPENCL_PROGRAMS_FILES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} opencl_programs/*)
  set(RESOURCES_FILES ${RESOURCES_FILES} ${OPENCL_PROGRAMS_FILES})
endif()

# Server, also install server subdirectory
set(SERVER_DEST_RESOURCE_FILES "")
if (WITH_SERVER)
  file(GLOB_RECURSE SERVER_RESOURCE_FILES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} server/*)
  add_qgis_resources("${CMAKE_CURRENT_SOURCE_DIR}" resources SERVER_DEST_RESOURCE_FILES "${SERVER_RESOURCE_FILES}")
endif()

# Server landingpage webapp
if (WITH_SERVER_LANDINGPAGE_WEBAPP)
  find_program(YARN yarn REQUIRED)

  file(GLOB_RECURSE LANDINGPAGE_SOURCE_FILES RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} server/src/landingpage/*)
  set(LANDINGPAGE_OUTPUT_PATH ${CMAKE_BINARY_DIR}/output/data/resources/server/src/landingpage)

  add_custom_target (server_landingpage ALL DEPENDS ${LANDINGPAGE_OUTPUT_PATH}/landingpage.stamp)

  # add_custom_command does not support OUTPUT_QUIET, so we need to make a cmake subcommand first:
  file(WRITE ${CMAKE_BINARY_DIR}/yarn_commands.cmake
  "execute_process(
      COMMAND ${YARN} install --frozen-lockfile
      WORKING_DIRECTORY ${LANDINGPAGE_OUTPUT_PATH}
      OUTPUT_QUIET
      ERROR_FILE ${CMAKE_BINARY_DIR}/yarn_error.log
      RESULT_VARIABLE YARN_RESULT
  )
  if(NOT YARN_RESULT EQUAL 0)
      file(READ ${CMAKE_BINARY_DIR}/yarn_error.log ERROR_CONTENTS)
      message(FATAL_ERROR \${ERROR_CONTENTS})
  endif()

  execute_process(
      COMMAND ${YARN} build
      WORKING_DIRECTORY ${LANDINGPAGE_OUTPUT_PATH}
      OUTPUT_QUIET
      ERROR_FILE ${CMAKE_BINARY_DIR}/yarn_error.log
      RESULT_VARIABLE YARN_RESULT
  )
  if(NOT YARN_RESULT EQUAL 0)
      file(READ ${CMAKE_BINARY_DIR}/yarn_error.log ERROR_CONTENTS)
      message(FATAL_ERROR \${ERROR_CONTENTS})
  endif()
  ")
  add_custom_command(
    POST_BUILD
    OUTPUT ${LANDINGPAGE_OUTPUT_PATH}/landingpage.stamp
    DEPENDS ${LANDINGPAGE_SOURCE_FILES}
    COMMAND ${CMAKE_COMMAND} -E copy_directory
    ${CMAKE_CURRENT_SOURCE_DIR}/server/src/landingpage/
    ${LANDINGPAGE_OUTPUT_PATH}/
    COMMAND ${CMAKE_COMMAND} -E touch ${LANDINGPAGE_OUTPUT_PATH}/landingpage.stamp
    COMMAND ${CMAKE_COMMAND} -P ${CMAKE_BINARY_DIR}/yarn_commands.cmake
  )
endif()

# Apple specific handling
if (APPLE)
  # ASN.1 definition files of PKIX elements
  set(RESOURCES_FILES ${RESOURCES_FILES} pkcs8.asn)
endif()

ADD_QGIS_RESOURCES("${CMAKE_CURRENT_SOURCE_DIR}" resources DEST_RESOURCE_FILES "${RESOURCES_FILES}")

set(SRSDB srs6.db)

add_custom_command(
  OUTPUT "${CMAKE_CURRENT_BINARY_DIR}/srs.db"
  COMMAND ${CMAKE_COMMAND} -E copy
    "${CMAKE_CURRENT_SOURCE_DIR}/${SRSDB}"
    "${CMAKE_CURRENT_BINARY_DIR}/srs.db"
  DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/${SRSDB}"
)
list(APPEND DEST_RESOURCE_FILES
  "${CMAKE_CURRENT_BINARY_DIR}/srs.db")
install(FILES ${SRSDB} DESTINATION ${QGIS_DATA_DIR}/resources RENAME srs.db)

list(APPEND DEST_RESOURCE_FILES ${SERVER_DEST_RESOURCE_FILES})
add_custom_target(resources ALL DEPENDS ${DEST_RESOURCE_FILES})
