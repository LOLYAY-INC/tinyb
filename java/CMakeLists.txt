find_package(Java REQUIRED)
include(UseJava)

if (NOT DEFINED $ENV{JAVA_HOME_NATIVE})
  set (JAVA_HOME_NATIVE $ENV{JAVA_HOME})
  set (JAVAC $ENV{JAVA_HOME}/bin/javac)
  set (JAR $ENV{JAVA_HOME}/bin/jar)
else ()
  set (JAVAC $ENV{JAVA_HOME_NATIVE}/bin/javac)
  set (JAR $ENV{JAVA_HOME_NATIVE}/bin/jar)
endif ()


set(CMAKE_JNI_TARGET TRUE)

# Create JNI output directory
set(JNI_OUTPUT_DIR "${CMAKE_CURRENT_BINARY_DIR}/jni")
file(MAKE_DIRECTORY ${JNI_OUTPUT_DIR})

file(GLOB JAVA_SOURCES "*.java")
add_jar(tinybjar ${JAVA_SOURCES}
                  MANIFEST ${CMAKE_CURRENT_BINARY_DIR}/manifest.txt
                  OUTPUT_NAME tinyb
)

# Generate JNI headers during Java compilation using javac -h (modern approach for Java 10+)
# This is done as part of add_jar, but we need to ensure the -h flag is used
add_custom_command(TARGET tinybjar
  PRE_BUILD
  COMMAND ${CMAKE_COMMAND} -E echo "Generating JNI headers with javac -h..."
  COMMAND ${Java_JAVAC_EXECUTABLE} -h ${JNI_OUTPUT_DIR} -d ${CMAKE_CURRENT_BINARY_DIR}/classes ${JAVA_SOURCES}
  WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
  COMMENT "Generating JNI headers"
)

set(JNI_HEADER_PATH "${JNI_OUTPUT_DIR}")
install (FILES ${CMAKE_CURRENT_BINARY_DIR}/tinyb.jar DESTINATION ${CMAKE_INSTALL_LIBDIR}/../lib/java)

add_subdirectory (jni)
