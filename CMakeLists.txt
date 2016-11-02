cmake_minimum_required(VERSION 2.4)

configure_file(hrpsysjy.in ${CMAKE_CURRENT_BINARY_DIR}/hrpsysjy)
configure_file(rtc.conf.in ${CMAKE_CURRENT_BINARY_DIR}/rtc.conf)

set(jarfile ${CMAKE_CURRENT_BINARY_DIR}/openrtm.jar)

set(IDLJ idlj)
set(idlj_flags -DORBIT2_IDL -fclient -fserver -emitAll -td src)
set(OPENRTM_IDL_DIR /usr/include/openrtm-1.1/rtm/idl)
set(JAVAC javac)
if (NOT DEFINED Java_VERSION_MAJOR)
  execute_process(COMMAND ${JAVAC} -version ERROR_VARIABLE jvar ERROR_STRIP_TRAILING_WHITESPACE)
  STRING( REGEX REPLACE "javac " "" Java_VERSION_STRING "${jvar}" )
  # following regex pulled from cmake-2.8 FindJava.cmake
  STRING( REGEX REPLACE "([0-9]+).*"          "\\1" Java_VERSION_MAJOR "${Java_VERSION_STRING}" )
  STRING( REGEX REPLACE "[0-9]+\\.([0-9]+).*" "\\1" Java_VERSION_MINOR "${Java_VERSION_STRING}" )
endif()
set(javac_flags -target ${Java_VERSION_MAJOR}.${Java_VERSION_MINOR} -d . -sourcepath src)
set(JAR jar)

add_custom_command(
  OUTPUT ${jarfile}
  COMMAND ${IDLJ} ${idlj_flags} -I. -I${OPENRTM_IDL_DIR} ${OPENRTM_IDL_DIR}/Manager.idl
  COMMAND ${IDLJ} ${idlj_flags} -I. -I${OPENRTM_IDL_DIR} ${OPENRTM_IDL_DIR}/OpenRTM.idl
  COMMAND ${IDLJ} ${idlj_flags} -I. -I${OPENRTM_IDL_DIR} ${OPENRTM_IDL_DIR}/DataPort.idl
  COMMAND ${IDLJ} ${idlj_flags} -I. -I${OPENRTM_IDL_DIR} ${OPENRTM_IDL_DIR}/InterfaceDataTypes.idl
  COMMAND ${JAVAC} ${javac_flags} src/*/*.java
  COMMAND ${JAR} cf ${jarfile} OpenRTM RTC RTM _SDOPackage
)

add_custom_target(OpenRTMStub ALL DEPENDS ${jarfile})

install(PROGRAMS ${CMAKE_CURRENT_BINARY_DIR}/hrpsysjy DESTINATION bin)
#install(FILES ${jarfile} DESTINATION share/java)
install(FILES ${jarfile} DESTINATION share/hrpsys/jar)
install(FILES rtm.py waitInput.py poseEditor.py DESTINATION share/hrpsys/jython)
install(FILES sample.py NullComponent.conf ${CMAKE_CURRENT_BINARY_DIR}/rtc.conf DESTINATION share/hrpsys/samples)
