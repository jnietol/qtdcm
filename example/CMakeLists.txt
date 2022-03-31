cmake_minimum_required(VERSION 3.0.2)

project(dicom)

if(NOT MSVC)
  add_definitions( -Wno-deprecated -DHAVE_CONFIG_H -fPIC )
endif()

set(CMAKE_MODULE_PATH ${CMAKE_MODULE_PATH} "${${PROJECT_NAME}_SOURCE_DIR}")

find_package( QT NAMES Qt6 Qt5 REQUIRED COMPONENTS Widgets Network )
find_package( Qt${QT_VERSION_MAJOR} REQUIRED COMPONENTS Core Widgets Network )
if (QT_VERSION_MAJOR EQUAL 6)
  find_package( Qt6 REQUIRED COMPONENTS Core5Compat )
endif()
get_target_property(QT${QT_VERSION_MAJOR}_QMAKE_EXECUTABLE Qt${QT_VERSION_MAJOR}::qmake LOCATION)
get_filename_component(QT${QT_VERSION_MAJOR}_BINARY_DIR ${QT${QT_VERSION_MAJOR}_QMAKE_EXECUTABLE} PATH)

find_package( QtDCM REQUIRED )
find_package( ITK REQUIRED )
if(ITK_FOUND)
  include(${ITK_USE_FILE})
endif(ITK_FOUND)

set(${PROJECT_NAME}_SRCS
  main.cpp
  dicom.cpp
)

set(${PROJECT_NAME}_HDRS
  dicom.h
)

set(${PROJECT_NAME}_UIS
  dicom.ui
)

set(${PROJECT_NAME}_MOC_HDRS ${${PROJECT_NAME}_HDRS})

if (QT_VERSION_MAJOR EQUAL 6)
qt6_wrap_ui( ${PROJECT_NAME}_UI_HDRS ${${PROJECT_NAME}_UIS})
qt6_wrap_cpp( ${PROJECT_NAME}_MOC_SRCS ${${PROJECT_NAME}_MOC_HDRS})
else()
qt5_wrap_ui( ${PROJECT_NAME}_UI_HDRS ${${PROJECT_NAME}_UIS})
qt5_wrap_cpp( ${PROJECT_NAME}_MOC_SRCS ${${PROJECT_NAME}_MOC_HDRS})
endif()

set(QT_LIBRARIES
  Qt${QT_VERSION_MAJOR}::Core
  Qt${QT_VERSION_MAJOR}::Widgets
  Qt${QT_VERSION_MAJOR}::Network
)

if (QT_VERSION_MAJOR EQUAL 6)
  list(APPEND QT_LIBRARIES Qt${QT_VERSION_MAJOR}::Core5Compat)
endif()

if(WIN32)
  list(APPEND QT_LIBRARIES Qt${QT_VERSION_MAJOR}::WinMain)
endif()

set(INCLUDE_DIRECTORIES
  ${QTDCM_INCLUDE_DIRS}
  ${QTDCM_BINARY_DIR}
  ${${PROJECT_NAME}_BINARY_DIR}
  ${${PROJECT_NAME}_SOURCE_DIR}
)
include_directories(${INCLUDE_DIRECTORIES})

add_executable(${PROJECT_NAME} WIN32 
               ${${PROJECT_NAME}_HDRS}  
               ${${PROJECT_NAME}_SRCS} 
               ${${PROJECT_NAME}_MOC_HDRS} 
               ${${PROJECT_NAME}_MOC_SRCS} 
               ${${PROJECT_NAME}_UI_HDRS})

target_link_libraries( ${PROJECT_NAME}
  ${QTDCM_LIBRARIES}
  ${QT_LIBRARIES}
)

if(APPLE)
  include(FixDCMTKMacLink)
  message("${QTDCM_DIR}")
  FixDCMTKMacExeLink(dicom ${QTDCM_BINARY_DIR})
endif()

install(TARGETS dicom RUNTIME DESTINATION bin)
