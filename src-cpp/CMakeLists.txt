set(LIBVER 1)

set(
    sources
    ConfImpl.cpp
    ConsumerImpl.cpp
    HandleImpl.cpp
    HeadersImpl.cpp
    KafkaConsumerImpl.cpp
    MessageImpl.cpp
    MetadataImpl.cpp
    ProducerImpl.cpp
    QueueImpl.cpp
    RdKafka.cpp
    TopicImpl.cpp
    TopicPartitionImpl.cpp
)

if(RDKAFKA_BUILD_STATIC)
  set(CMAKE_POSITION_INDEPENDENT_CODE ON)
  set(RDKAFKA_BUILD_MODE STATIC)
else()
  set(RDKAFKA_BUILD_MODE SHARED)
endif()

add_library(rdkafka++ ${RDKAFKA_BUILD_MODE} ${sources})
if(NOT RDKAFKA_BUILD_STATIC)
  set_property(TARGET rdkafka++ PROPERTY SOVERSION ${LIBVER})
endif()

target_link_libraries(rdkafka++ PUBLIC rdkafka)

# Support '#include <rdkafcpp.h>'
target_include_directories(rdkafka++ PUBLIC "$<BUILD_INTERFACE:${CMAKE_CURRENT_LIST_DIR}>")
if(NOT RDKAFKA_BUILD_STATIC)
    target_compile_definitions(rdkafka++ PRIVATE LIBRDKAFKACPP_EXPORTS)
    if(WIN32)
        target_compile_definitions(rdkafka++ INTERFACE LIBRDKAFKACPP_EXPORTS=0)
    endif()
endif()

# Generate pkg-config file
set(PKG_CONFIG_NAME
    "librdkafka++"
)
set(PKG_CONFIG_DESCRIPTION
    "The Apache Kafka C/C++ library"
)
set(PKG_CONFIG_VERSION
    "${PROJECT_VERSION}"
)
set(PKG_CONFIG_REQUIRES "rdkafka")
set(PKG_CONFIG_CFLAGS
    "-I\${includedir}"
)
set(PKG_CONFIG_LIBS
    "-L\${libdir} -lrdkafka++"
)
set(PKG_CONFIG_LIBS_PRIVATE
    "-lrdkafka -lstdc++"
)
configure_file(
    "../packaging/cmake/rdkafka.pc.in"
    "${GENERATED_DIR}/rdkafka++.pc"
    @ONLY
)
install(FILES ${GENERATED_DIR}/rdkafka++.pc
    DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig"
)
if(RDKAFKA_BUILD_STATIC)
  set(PKG_CONFIG_NAME
    "librdkafka++-static"
  )
  set(PKG_CONFIG_DESCRIPTION
    "The Apache Kafka C/C++ library (static)"
  )
  set(PKG_CONFIG_LIBS
    "-L\${libdir} \${libdir}/librdkafka++.a"
  )
  configure_file(
    "../packaging/cmake/rdkafka.pc.in"
    "${GENERATED_DIR}/rdkafka++-static.pc"
    @ONLY
  )
  install(FILES ${GENERATED_DIR}/rdkafka.pc
    DESTINATION "${CMAKE_INSTALL_LIBDIR}/pkgconfig"
  )
endif()

install(
    TARGETS rdkafka++
    EXPORT "${targets_export_name}"
    LIBRARY DESTINATION "${CMAKE_INSTALL_LIBDIR}"
    ARCHIVE DESTINATION "${CMAKE_INSTALL_LIBDIR}"
    RUNTIME DESTINATION "${CMAKE_INSTALL_BINDIR}"
    INCLUDES DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}"
)

install(
    FILES "rdkafkacpp.h"
    DESTINATION "${CMAKE_INSTALL_INCLUDEDIR}/librdkafka"
)
