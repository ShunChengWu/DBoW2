cmake_minimum_required(VERSION 3.10)
cmake_policy(SET CMP0048 NEW)
project(DBoW2 VERSION 1)
set(CMAKE_CXX_STANDARD 17)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS}  -Wall  -O3 -march=native ")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wall  -O3 -march=native")

set(HDRS_DBOW2
  DBoW2/BowVector.h
  DBoW2/FORB.h 
  DBoW2/FClass.h       
  DBoW2/FeatureVector.h
  DBoW2/ScoringObject.h   
  DBoW2/TemplatedVocabulary.h)
set(SRCS_DBOW2
  DBoW2/BowVector.cpp
  DBoW2/FORB.cpp      
  DBoW2/FeatureVector.cpp
  DBoW2/ScoringObject.cpp)

set(HDRS_DUTILS
  DUtils/Random.h
  DUtils/Timestamp.h)
set(SRCS_DUTILS
  DUtils/Random.cpp
  DUtils/Timestamp.cpp)

find_package(OpenCV 4.0 QUIET)
if(NOT OpenCV_FOUND)
   find_package(OpenCV 2.4.3 QUIET)
   if(NOT OpenCV_FOUND)
      message(FATAL_ERROR "OpenCV > 2.4.3 not found.")
   endif()
endif()

set(LIBRARY_OUTPUT_PATH ${PROJECT_SOURCE_DIR}/lib)

include_directories(${OpenCV_INCLUDE_DIRS})

#
# Deployment
#
include(GNUInstallDirs)
SET(targetname DBoW2)
SET(${targetname}_install_cmake_dir ${CMAKE_INSTALL_LIBDIR}/cmake/DBoW2)
set(${targetname}_install_include_dir       ${CMAKE_INSTALL_INCLUDEDIR})
set(${targetname}_config_dir                ${CMAKE_CURRENT_BINARY_DIR}/cmake/)
set(${targetname}_config                    ${${targetname}_config_dir}/${targetname}Config.cmake)
set(${targetname}_config_version            ${${targetname}_config_dir}/${targetname}ConfigVersion.cmake)


#MESSAGE("${targetname}_install_cmake_dir: " ${${targetname}_install_cmake_dir})
#MESSAGE("${targetname}_install_include_dir: " ${${targetname}_install_include_dir})
#MESSAGE("${targetname}_config_dir: " ${targetname}_config_dir)
#MESSAGE("${targetname}_config: " ${targetname}_config)
#MESSAGE("${targetname}_config_version: " ${${targetname}_config_version})

add_library(DBoW2 SHARED ${SRCS_DBOW2} ${SRCS_DUTILS})
target_link_libraries(DBoW2 ${OpenCV_LIBS})
target_include_directories(DBoW2 
	PUBLIC $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}>
	PUBLIC $<INSTALL_INTERFACE:${${targetname}_install_include_dir}>
)
add_library(DBoW2::DBoW2 ALIAS DBoW2)

include(CMakePackageConfigHelpers)
write_basic_package_version_file(
    ${DBoW2_config_version}
    VERSION ${DBoW2_VERSION}
    COMPATIBILITY SameMajorVersion
)
configure_package_config_file(
    cmake/DBoW2Config.cmake.in
    ${DBoW2_config}
    INSTALL_DESTINATION ${DBoW2_install_cmake_dir}
    PATH_VARS DBoW2_install_cmake_dir
)

# Install config files
install(DIRECTORY DESTINATION ${CMAKE_INSTALL_PREFIX})
install(FILES ${${targetname}_config} ${${targetname}_config_version}
    DESTINATION ${${targetname}_install_cmake_dir}
)

# Install header files
install(
    FILES ${HDRS_DBOW2} 
    DESTINATION "${${targetname}_install_include_dir}/DBoW2/"
)
install(
    FILES ${HDRS_DUTILS} 
    DESTINATION "${${targetname}_install_include_dir}/DUtils/"
)

# Export targets and install them
install(TARGETS ${targetname}
    EXPORT ${targetname}Targets
    INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
)
install(EXPORT ${targetname}Targets
    FILE ${targetname}Targets.cmake
    DESTINATION ${${targetname}_install_cmake_dir}
    NAMESPACE ${targetname}::
)

CONFIGURE_FILE(
  "${PROJECT_SOURCE_DIR}/cmake/cmake_uninstall.cmake.in"
  "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake"
  IMMEDIATE @ONLY)

ADD_CUSTOM_TARGET(uninstall
  "${CMAKE_COMMAND}" -P "${CMAKE_CURRENT_BINARY_DIR}/cmake_uninstall.cmake")

