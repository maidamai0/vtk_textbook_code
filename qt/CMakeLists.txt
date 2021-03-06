set(CMAKE_AUTOMOC ON)
set(CMAKE_AUTOUIC ON)

set(QT_LIBS Widgets Core Gui Quick Qml)
find_package(Qt5 COMPONENTS ${QT_LIBS} REQUIRED)
foreach(QT_LIB ${QT_LIBS})
    link_libraries(Qt5::${QT_LIB})
endforeach()

get_target_property(_qmake_executable Qt5::qmake IMPORTED_LOCATION)
get_filename_component(_qt_bin_dir "${_qmake_executable}" DIRECTORY)
find_program(WINDEPLOYQT_EXECUTABLE windeployqt HINTS "${_qt_bin_dir}")

# every sub directory is a qt based project
file(GLOB subdirs RELATIVE ${CMAKE_CURRENT_SOURCE_DIR} ${CMAKE_CURRENT_SOURCE_DIR}/*)
foreach(subdir ${subdirs})
    if(IS_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/${subdir})
        file(GLOB src "${subdir}/*.cpp")
        file(GLOB hdr "${subdir}/*.h")
        file(GLOB ui "${subdir}/*.ui")
        add_executable(${subdir} ${ui} ${src} ${hdr})

        # link with vtk
        if (VTK_VERSION VERSION_LESS "8.90.0")
            # old system
            include(${VTK_USE_FILE})
            target_link_libraries(${subdir} PRIVATE ${VTK_LIBRARIES})
        else ()
            # include all components
            target_link_libraries(${subdir} PRIVATE ${VTK_LIBRARIES})
            # vtk_module_autoinit is needed
            vtk_module_autoinit(
                TARGETS ${subdir}
                MODULES ${VTK_LIBRARIES}
                )
        endif () 

        # copy qt runtime
        add_custom_command(
            TARGET ${subdir}
            POST_BUILD
            COMMAND ${WINDEPLOYQT_EXECUTABLE} --qmldir ${CMAKE_CURRENT_SOURCE_DIR}/${subdir}/ $<TARGET_FILE:${subdir}>
        )
    endif()
endforeach(subdir )