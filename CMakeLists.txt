cmake_minimum_required(VERSION 3.15)
cmake_policy(SET CMP0091 NEW)
project(cimgui)

set (CMAKE_CXX_STANDARD 11)

if (EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/cimgui/imgui/imgui_tables.cpp)
	set(TABLES_SOURCE "cimgui/imgui/imgui_tables.cpp")
else()
	set(TABLES_SOURCE "")
endif()


#general settings
file(GLOB IMGUI_SOURCES
    cimgui/cimgui.h
    cimgui/imgui/imgui.h
    cimgui/cimgui.cpp
    cimgui/imgui/imgui.cpp
    cimgui/imgui/imgui_draw.cpp
    cimgui/imgui/imgui_demo.cpp
    cimgui/imgui/imgui_widgets.cpp
	${TABLES_SOURCE}

    cimguizmo/cimguizmo.h
	cimguizmo/cimguizmo.cpp
    cimguizmo/ImGuizmo/GraphEditor.h
    cimguizmo/ImGuizmo/ImCurveEdit.h
    cimguizmo/ImGuizmo/ImGradient.h
    cimguizmo/ImGuizmo/ImGuizmo.h
    cimguizmo/ImGuizmo/ImSequencer.h
    cimguizmo/ImGuizmo/ImZoomSlider.h
    cimguizmo/ImGuizmo/GraphEditor.cpp
    cimguizmo/ImGuizmo/ImCurveEdit.cpp
    cimguizmo/ImGuizmo/ImGradient.cpp
    cimguizmo/ImGuizmo/ImGuizmo.cpp
    cimguizmo/ImGuizmo/ImSequencer.cpp

    cimnodes/cimnodes.h
	cimnodes/cimnodes.cpp
    cimnodes/imnodes/imnodes.h
    cimnodes/imnodes/imnodes_internal.h
    cimnodes/imnodes/imnodes.cpp

    cimplots/cimplot.h
    cimplots/cimplot.cpp
	cimplots/implot/implot.cpp
	cimplots/implot/implot_demo.cpp
	cimplots/implot/implot_items.cpp
)

set(IMGUI_STATIC "no" CACHE STRING "Build as a static library")
set(IMGUI_FREETYPE "no" CACHE STRING "Build with freetype library")
set(IMGUI_LIBRARIES )

if(IMGUI_FREETYPE)
	message(STATUS "Building with freetype")
	FIND_PACKAGE(freetype REQUIRED PATHS ${FREETYPE_PATH})
	list(APPEND IMGUI_LIBRARIES freetype)
	list(APPEND IMGUI_SOURCES imgui/misc/freetype/imgui_freetype.cpp)
endif(IMGUI_FREETYPE)

#add library and link
if (IMGUI_STATIC)
    add_library(cimgui STATIC ${IMGUI_SOURCES})
else (IMGUI_STATIC)
    add_library(cimgui SHARED ${IMGUI_SOURCES})
endif (IMGUI_STATIC)

if(IMGUI_FREETYPE)
	target_compile_definitions(cimgui PUBLIC IMGUI_ENABLE_FREETYPE)
	target_compile_definitions(cimgui PUBLIC IMGUI_ENABLE_STB_TRUETYPE)
endif(IMGUI_FREETYPE)

if(IMGUI_WCHAR32)
	target_compile_definitions(cimgui PUBLIC IMGUI_USE_WCHAR32)
endif(IMGUI_WCHAR32)

add_definitions("-DIMGUI_USER_CONFIG=\"../cimconfig.h\"")
target_compile_definitions(cimgui PUBLIC IMGUI_DISABLE_OBSOLETE_FUNCTIONS=1)
target_compile_definitions(cimgui PUBLIC IMNODES_NAMESPACE=imnodes)
if (WIN32)
    target_compile_definitions(cimgui PUBLIC IMGUI_IMPL_API=extern\t\"C\"\t__declspec\(dllexport\))
    list(APPEND IMGUI_LIBRARIES imm32)
else (WIN32)
    target_compile_definitions(cimgui PUBLIC IMGUI_IMPL_API=extern\t\"C\"\t)
endif (WIN32)

target_include_directories(cimgui PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/cimgui)
target_include_directories(cimgui PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/cimgui/imgui)
target_include_directories(cimgui PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/cimguizmo)
target_include_directories(cimgui PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/cimguizmo/ImGuizmo)
target_include_directories(cimgui PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/cimnodes)
target_include_directories(cimgui PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/cimnodes/imnodes)
target_include_directories(cimgui PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/cimplot)
target_include_directories(cimgui PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/cimplot/implot)
set_target_properties(cimgui PROPERTIES PREFIX "")
target_link_libraries(cimgui ${IMGUI_LIBRARIES})

#install
install(TARGETS cimgui
    RUNTIME DESTINATION  .
    LIBRARY DESTINATION  .
    ARCHIVE DESTINATION  .
)

#test
set(CIMGUI_TEST "no" CACHE STRING "Enable compilation of a test unit based on imgui null")

if (CIMGUI_TEST)
  add_subdirectory(test)
endif ()
