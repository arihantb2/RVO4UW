cmake_minimum_required(VERSION 3.5)
project(RVO4UW VERSION 0.0.0)

IF(NOT CMAKE_BUILD_TYPE)
  SET(CMAKE_BUILD_TYPE Release)
ENDIF()

MESSAGE("Build type: " ${CMAKE_BUILD_TYPE})

set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -O3 -march=native")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3 -march=native")

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)
set(CMAKE_CXX_EXTENSIONS OFF)

include(GNUInstallDirs)
include(CMakePackageConfigHelpers)

set(CMAKE_ARCHIVE_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_LIBRARY_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/lib)
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)
set(CMAKE_BUILD_RPATH "${CMAKE_LIBRARY_OUTPUT_DIRECTORY}")
set(CMAKE_INSTALL_RPATH "${CMAKE_INSTALL_PREFIX}/${CMAKE_INSTALL_LIBDIR}")
set(CMAKE_INSTALL_RPATH_USE_LINK_PATH TRUE)

find_package(OpenCV QUIET)
if(NOT OpenCV_FOUND)
   find_package(OpenCV 2.4.3 QUIET)
   if(NOT OpenCV_FOUND)
      message(FATAL_ERROR "OpenCV > 2.4.3 not found.")
   endif()
endif()

find_package(Eigen3 3.1.0 CONFIG REQUIRED)
find_package(Pangolin REQUIRED)
find_package(opengv CONFIG REQUIRED)

option(BUILD_EXAMPLES "Build example executables" OFF)

add_subdirectory(Thirdparty/DBoW2)
add_subdirectory(Thirdparty/g2o)

add_library(${PROJECT_NAME} SHARED
   src/System.cc
   src/Tracking.cc
   src/LocalMapping.cc
   src/LoopClosing.cc
   src/ORBextractor.cc
   src/ORBmatcher.cc
   src/FrameDrawer.cc
   src/Converter.cc
   src/MapPoint.cc
   src/KeyFrame.cc
   src/Map.cc
   src/MapDrawer.cc
   src/Optimizer.cc
   src/PnPsolver.cc
   src/Frame.cc
   src/KeyFrameDatabase.cc
   src/Sim3Solver.cc
   src/Initializer.cc
   src/Viewer.cc
   src/suace.cpp
)

set(RVO4UW_NAMESPACED_INCLUDE_DIR ${CMAKE_CURRENT_BINARY_DIR}/include/RVO4UW)
file(MAKE_DIRECTORY ${RVO4UW_NAMESPACED_INCLUDE_DIR})
file(GLOB RVO4UW_PUBLIC_HEADERS "${PROJECT_SOURCE_DIR}/include/*.h")
foreach(header ${RVO4UW_PUBLIC_HEADERS})
  get_filename_component(header_name ${header} NAME)
  configure_file(${header} ${RVO4UW_NAMESPACED_INCLUDE_DIR}/${header_name} COPYONLY)
endforeach()

target_include_directories(${PROJECT_NAME}
PUBLIC
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}>
  $<BUILD_INTERFACE:${PROJECT_SOURCE_DIR}/include>
  $<BUILD_INTERFACE:${CMAKE_CURRENT_BINARY_DIR}/include>
  ${EIGEN3_INCLUDE_DIR}
  ${Pangolin_INCLUDE_DIRS}
  $<INSTALL_INTERFACE:${CMAKE_INSTALL_INCLUDEDIR}>
)

target_link_libraries(${PROJECT_NAME}
   ${OpenCV_LIBS}
   ${EIGEN3_LIBS}
   ${Pangolin_LIBRARIES}
   DBoW2
   g2o
   opengv
)

# Install and export
install(TARGETS ${PROJECT_NAME}
  EXPORT RVO4UWTargets
  ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
  LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
  RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
)

install(DIRECTORY ${CMAKE_CURRENT_BINARY_DIR}/include/RVO4UW
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
  FILES_MATCHING PATTERN "*.h"
)
install(DIRECTORY Thirdparty/DBoW2/DBoW2 Thirdparty/DBoW2/DUtils
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/Thirdparty/DBoW2
  FILES_MATCHING PATTERN "*.h"
)
install(DIRECTORY Thirdparty/g2o/g2o
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/Thirdparty/g2o
  FILES_MATCHING PATTERN "*.h" PATTERN "*.hpp"
)
install(FILES ${CMAKE_CURRENT_BINARY_DIR}/Thirdparty/g2o/config.h
  DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}/Thirdparty/g2o
)

export(EXPORT RVO4UWTargets
  FILE "${CMAKE_CURRENT_BINARY_DIR}/RVO4UWTargets.cmake"
  NAMESPACE RVO4UW::
)

configure_package_config_file(
  ${PROJECT_SOURCE_DIR}/cmake/RVO4UWConfig.cmake.in
  ${CMAKE_CURRENT_BINARY_DIR}/RVO4UWConfig.cmake
  INSTALL_DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/RVO4UW
)

write_basic_package_version_file(
  ${CMAKE_CURRENT_BINARY_DIR}/RVO4UWConfigVersion.cmake
  VERSION ${PROJECT_VERSION}
  COMPATIBILITY SameMajorVersion
)

install(EXPORT RVO4UWTargets
  FILE RVO4UWTargets.cmake
  NAMESPACE RVO4UW::
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/RVO4UW
)

install(FILES
  ${CMAKE_CURRENT_BINARY_DIR}/RVO4UWConfig.cmake
  ${CMAKE_CURRENT_BINARY_DIR}/RVO4UWConfigVersion.cmake
  DESTINATION ${CMAKE_INSTALL_LIBDIR}/cmake/RVO4UW
)

if(BUILD_EXAMPLES)
  add_executable(stereo_kitti
    Examples/Stereo/stereo_kitti.cc)
  target_link_libraries(stereo_kitti ${PROJECT_NAME})

  add_executable(stereo_euroc
    Examples/Stereo/stereo_euroc.cc)
  target_link_libraries(stereo_euroc ${PROJECT_NAME})
endif()

