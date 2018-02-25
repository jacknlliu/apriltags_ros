# swatbotics/apriltags-cpp/CMakeLists.txt

cmake_minimum_required(VERSION 2.8)

project(apriltagscpp)

# set(EXECUTABLE_OUTPUT_PATH ${CMAKE_BINARY_DIR})
# set(LIBRARY_OUTPUT_PATH ${CMAKE_BINARY_DIR})
find_package(catkin REQUIRED COMPONENTS
            cmake_modules
            roscpp
)

catkin_package(
  INCLUDE_DIRS include
  CATKIN_DEPENDS roscpp
  LIBRARIES ${PROJECT_NAME}
)

# find_package(OpenCV REQUIRED)
find_package(Eigen3 REQUIRED)
include_directories(${Eigen_INCLUDE_DIRS})

message("here: ${Eigen_INCLUDE_DIRS}")

if(CMAKE_VERSION VERSION_GREATER "2.8.11")
  cmake_policy(SET CMP0022 OLD)
endif()

if(APPLE)
  include_directories(/opt/local/include) # MacPorts
  link_directories(/opt/local/lib)
  find_library(OPENGL_LIBRARY OpenGL)
else()
  find_library(OPENGL_LIBRARY GL)
  find_library(GLU_LIBRARY GLU)
  set(OPENGL_LIBRARY ${OPENGL_LIBRARY} ${GLU_LIBRARY})
endif()

find_library(GLUT_LIBRARY glut)

include(FindPkgConfig)

find_package(OpenCV REQUIRED)

include_directories(${OPENCV_INCLUDE_DIRS}
${catkin_INCLUDE_DIRS})


pkg_check_modules(CAIRO cairo)

if (${CAIRO_FOUND})
add_definitions(-DMZ_HAVE_CAIRO)
endif ()

find_package( CGAL QUIET COMPONENTS  )
if (CGAL_FOUND)
  include( ${CGAL_USE_FILE} )
  add_definitions(-DHAVE_CGAL)
  find_package( Boost REQUIRED )
else()
  message("CGAL not found, can not use new quad detector")
endif()

set(CMAKE_C_FLAGS "-Wall -g")
set(CMAKE_CXX_FLAGS "-Wall -g")

if (APPLE OR UNIX)
set(CMAKE_C_FLAGS "${CMAKE_C_FLAGS} -Wsign-compare -frounding-math")
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -Wsign-compare -frounding-math")
endif()

set(CMAKE_C_FLAGS_DEBUG "-O")
set(CMAKE_CXX_FLAGS_DEBUG "-O")

set(CMAKE_C_FLAGS_RELEASE "-O2")
set(CMAKE_CXX_FLAGS_RELEASE "-O2")

include_directories( include/ )

# set(INCLUDE_INSTALL_DIR "include")
# set(LIBRARY_INSTALL_DIR "lib")
# set(CONFIG_INSTALL_DIR "${LIBRARY_INSTALL_DIR}/${PROJECT_NAME}/cmake")

add_subdirectory(src)

# Install the package.xml file (to satisfy apriltagscpp).
# install(FILES package.xml
#   DESTINATION share/${PROJECT_NAME}
# )

include(CMakePackageConfigHelpers)
# configure_package_config_file("cmake/${PROJECT_NAME}Config.cmake.in"
#   "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
#   INSTALL_DESTINATION "${CONFIG_INSTALL_DIR}"
#   PATH_VARS INCLUDE_INSTALL_DIR)
# install(FILES "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake"
#   DESTINATION "${CONFIG_INSTALL_DIR}")

add_library(${PROJECT_NAME} SHARED
  src/CameraUtil.cpp
  src/DebugImage.cpp
  src/Geometry.cpp
  src/GrayModel.cpp
  src/MathUtil.cpp
  src/Refine.cpp
  src/TagDetector.cpp
  src/TagFamily.cpp
  src/TagFamilies.cpp
  src/UnionFindSimple.cpp
)

set(AT_LIBS ${PROJECT_NAME} ${OpenCV_LIBRARIES} ${Eigen_LIBRARIES})

target_link_libraries(${PROJECT_NAME} ${OpenCV_LIBRARIES})

add_executable(camtest src/camtest.cpp)
target_link_libraries(camtest ${AT_LIBS})

add_executable(tagtest src/tagtest.cpp)
target_link_libraries(tagtest ${AT_LIBS})

add_executable(quadtest src/quadtest.cpp)
target_link_libraries(quadtest ${AT_LIBS})

if (GLUT_LIBRARY)

  add_executable(gltest src/gltest.cpp)
  target_link_libraries(gltest ${GLUT_LIBRARY} ${OPENGL_LIBRARY} ${AT_LIBS})

endif()

if (CAIRO_FOUND)

  add_executable(maketags src/maketags.cpp)
  target_link_libraries(maketags ${CAIRO_LIBRARIES} ${AT_LIBS} ${CAIRO_LIBS})

endif()




install(DIRECTORY include
        DESTINATION ${CATKIN_PACKAGE_INCLUDE_DESTINATION})


install(TARGETS ${PROJECT_NAME}
        ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
        LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
        RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION})

message("include dir include/${PROJECT_NAME} to ${CATKIN_PACKAGE_INCLUDE_DESTINATION}")
