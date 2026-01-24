cmake_minimum_required(VERSION 3.5)
project(linefit_ground_segmentation)

find_package(catkin QUIET)

find_package(PCL REQUIRED)

find_package(Eigen3 REQUIRED)

IF(catkin_FOUND)

    catkin_package(
        INCLUDE_DIRS include ${EIGEN3_INCLUDE_DIR}
        LIBRARIES ${PROJECT_NAME}
        DEPENDS PCL
    )

ENDIF()

include_directories(
    include
    ${EIGEN3_INCLUDE_DIR}
    ${PCL_INCLUDE_DIRS}
)

add_library(${PROJECT_NAME} SHARED
    src/ground_segmentation.cc
    src/segment.cc
    src/bin.cc
    src/viewer.cc
)

target_link_libraries(${PROJECT_NAME} ${PCL_LIBRARIES})

target_compile_features(${PROJECT_NAME} PUBLIC cxx_std_14)

add_definitions(${PCL_DEFINITIONS})

target_include_directories(${PROJECT_NAME} PUBLIC
    $<BUILD_INTERFACE:${CMAKE_CURRENT_SOURCE_DIR}/include>
    $<INSTALL_INTERFACE:include>
)

IF(catkin_FOUND)

    install(TARGETS ${PROJECT_NAME}
        ARCHIVE DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
        LIBRARY DESTINATION ${CATKIN_PACKAGE_LIB_DESTINATION}
        RUNTIME DESTINATION ${CATKIN_GLOBAL_BIN_DESTINATION}
    )

    install(DIRECTORY include/
        DESTINATION ${CATKIN_GLOBAL_INCLUDE_DESTINATION}
        FILES_MATCHING PATTERN "*.h"
        PATTERN ".svn" EXCLUDE
    )

ELSE()
    include(GNUInstallDirs)
    include(CMakePackageConfigHelpers)

    install(TARGETS ${PROJECT_NAME}
        EXPORT ${PROJECT_NAME}Targets
        ARCHIVE DESTINATION ${CMAKE_INSTALL_LIBDIR}
        LIBRARY DESTINATION ${CMAKE_INSTALL_LIBDIR}
        RUNTIME DESTINATION ${CMAKE_INSTALL_BINDIR}
        INCLUDES DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
    )

    install(DIRECTORY include/
        DESTINATION ${CMAKE_INSTALL_INCLUDEDIR}
        FILES_MATCHING PATTERN "*.h"
    )

    set(CONFIG_INSTALL_DIR ${CMAKE_INSTALL_LIBDIR}/cmake/${PROJECT_NAME})
    set(generated_config "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}Config.cmake")
    set(version_config "${CMAKE_CURRENT_BINARY_DIR}/${PROJECT_NAME}ConfigVersion.cmake")
    set(config_in "${CMAKE_CURRENT_SOURCE_DIR}/cmake/${PROJECT_NAME}Config.cmake.in")

    write_basic_package_version_file(
        "${version_config}"
        VERSION 0.0.1
        COMPATIBILITY AnyNewerVersion
    )

    configure_package_config_file(
        "${config_in}"
        "${generated_config}"
        INSTALL_DESTINATION "${CONFIG_INSTALL_DIR}"
    )

    install(FILES ${generated_config} ${version_config}
        DESTINATION ${CONFIG_INSTALL_DIR}
    )

    install(EXPORT ${PROJECT_NAME}Targets
        NAMESPACE ${PROJECT_NAME}::
        DESTINATION ${CONFIG_INSTALL_DIR}
    )
ENDIF()


#############
# QTCREATOR #
#############
FILE(GLOB_RECURSE LibFiles "include/*")
add_custom_target(headers SOURCES ${LibFiles})
