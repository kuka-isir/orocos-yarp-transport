#
# The find_package macro for Orocos-RTT works best with
# cmake >= 2.6.3
#
cmake_minimum_required(VERSION 2.6.3)

#
# This creates a standard cmake project. You may extend this file with
# any cmake macro you see fit.
#
# The CMake 'target' names are identical to the first argument of the
# macro, except for orocos_typekit, where the target is fully
# controlled by generated code of 'typegen'.
#
project(rtt-yarp-transport)

# Look for deps
find_package(OROCOS-RTT REQUIRED)
find_package(YARP REQUIRED)
include_directories(${YARP_INCLUDE_DIRS})

# Defines the orocos_* cmake macros
include(${OROCOS-RTT_USE_FILE_PATH}/UseOROCOS-RTT.cmake)


# YARP transport
include_directories(transport/)
orocos_plugin( ${PROJECT_NAME} transport/YarpTransport.cpp )
target_link_libraries( ${PROJECT_NAME} ${OROCOS-RTT_LIBRARIES} ${YARP_LIBRARIES})

# YARP service
orocos_service( yarp-service service/Yarp-service.cpp )

# Test file
if(CATKIN_ENABLE_TESTING)
	orocos_executable(FooTest testing/FooTest.cpp)
	target_link_libraries(FooTest ${PROJECT_NAME})
	add_test(YarpTransport FooTest-${OROCOS_TARGET})
endif()

# Doc
option(BUILD_DOC "Build doc (requires docbook)")
if(BUILD_DOC)
	find_program(DOCBOOK docbook2pdf)
	if (DOCBOOK-NOTFOUND)
		message(STATUS "docbook2pdf not found, cannot build doc")
	else(DOCBOOK-NOTFOUND)
		add_custom_target(doc ALL ${DOCBOOK} ${CMAKE_SOURCE_DIR}/doc/yarp-transport-doc.xml)
		install(FILES ${CMAKE_BINARY_DIR}/yarp-transport-doc.pdf
			DESTINATION share/doc/yarp-transport)
	endif(DOCBOOK-NOTFOUND)
endif()

# Additional headers (not in typekit):
#
# Installs in the include/orocos/transports/ directory
#
orocos_install_headers( transport/YarpTransport.hpp
			transport/YarpTemplateProtocol.hpp
			transport/YarpChannelElement.hpp
			transport/yarp_bottle_archive.hpp
			)

#
# Generates and installs our package. Must be the last statement such
# that it can pick up all above settings.
#
orocos_generate_package(INCLUDE_DIRS transport)
