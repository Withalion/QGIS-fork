set(AUTH_PKCS12_SRCS
  core/qgsauthpkcs12method.cpp
)

set(AUTH_PKCS12_HDRS
  core/qgsauthpkcs12method.h
)

set(AUTH_PKCS12_UIS_H "")

if (WITH_GUI)
  set(AUTH_PKCS12_SRCS ${AUTH_PKCS12_SRCS}
    gui/qgsauthpkcs12edit.cpp
  )
  set(AUTH_PKCS12_HDRS ${AUTH_PKCS12_HDRS}
    gui/qgsauthpkcs12edit.h
  )
  set(AUTH_PKCS12_UIS gui/qgsauthpkcs12edit.ui)
  qt_wrap_ui(AUTH_PKCS12_UIS_H ${AUTH_PKCS12_UIS})
endif()

# static library
add_library(authmethod_pkcs12_a STATIC ${AUTH_PKCS12_SRCS} ${AUTH_PKCS12_HDRS} ${AUTH_PKCS12_UIS_H})

target_include_directories(authmethod_pkcs12_a PUBLIC ${CMAKE_SOURCE_DIR}/src/auth/pkipkcs12/core)

# require c++17
target_compile_features(authmethod_pkcs12_a PRIVATE cxx_std_17)

target_link_libraries(authmethod_pkcs12_a qgis_core)

if (WITH_GUI)
  target_include_directories(authmethod_pkcs12_a PRIVATE
    ${CMAKE_SOURCE_DIR}/src/auth/pkipkcs12/gui
    ${CMAKE_BINARY_DIR}/src/auth/pkipkcs12
  )
  target_link_libraries (authmethod_pkcs12_a qgis_gui)
endif()

target_compile_definitions(authmethod_pkcs12_a PRIVATE "-DQT_NO_FOREACH")



if (FORCE_STATIC_LIBS)
  # for (external) mobile apps to be able to pick up provider for linking
  install (TARGETS authmethod_pkcs12_a ARCHIVE DESTINATION ${QGIS_PLUGIN_DIR})
else()
  # dynamically loaded module
  add_library(authmethod_pkcs12 MODULE ${AUTH_PKCS12_SRCS} ${AUTH_PKCS12_HDRS} ${AUTH_PKCS12_UIS_H})

  # require c++17
  target_compile_features(authmethod_pkcs12 PRIVATE cxx_std_17)

  target_link_libraries(authmethod_pkcs12 qgis_core)

  if (WITH_GUI)
    target_include_directories(authmethod_pkcs12 PRIVATE
      ${CMAKE_SOURCE_DIR}/src/auth/pkipkcs12/gui
      ${CMAKE_BINARY_DIR}/src/auth/pkipkcs12
    )
    target_link_libraries (authmethod_pkcs12 qgis_gui)
    add_dependencies(authmethod_pkcs12 ui)
  endif()

  target_compile_definitions(authmethod_pkcs12 PRIVATE "-DQT_NO_FOREACH")

  install (TARGETS authmethod_pkcs12
    RUNTIME DESTINATION ${QGIS_PLUGIN_DIR}
    LIBRARY DESTINATION ${QGIS_PLUGIN_DIR}
  )
endif()
