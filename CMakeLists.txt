cmake_minimum_required(VERSION 3.15)
cmake_policy(SET CMP0091 NEW)
project(cimgui)

set (CMAKE_CXX_STANDARD 11)

if (EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/imgui/imgui_tables.cpp)
	set(TABLES_SOURCE "imgui/imgui_tables.cpp")
else()
	set(TABLES_SOURCE "")
endif()


#general settings
file(GLOB IMGUI_SOURCES
    cimgui.cpp
    imgui/imgui.cpp
    imgui/imgui_draw.cpp
    imgui/imgui_demo.cpp
    imgui/imgui_widgets.cpp
	${TABLES_SOURCE}
	imgui/backends/imgui_impl_sdl2.cpp
	imgui/backends/imgui_impl_sdlrenderer2.cpp
	imgui/backends/imgui_impl_glfw.cpp
	imgui/backends/imgui_impl_opengl2.cpp
	imgui/backends/imgui_impl_opengl3.cpp
	imgui/backends/imgui_impl_vulkan.cpp
)

set(IMGUI_LIBRARIES )
if(CMAKE_SYSTEM MATCHES Windows)
	list(APPEND IMGUI_SOURCES
		imgui/backends/imgui_impl_win32.cpp
		imgui/backends/imgui_impl_dx9.cpp
		imgui/backends/imgui_impl_dx10.cpp
		imgui/backends/imgui_impl_dx11.cpp
		imgui/backends/imgui_impl_dx12.cpp
	)
	
	list(APPEND IMGUI_LIBRARIES "dxgi.lib")
endif()

if(CMAKE_SYSTEM MATCHES Linux)
	#add linux specific libraries/backends
endif()


find_package(SDL2 REQUIRED)
list(APPEND IMGUI_LIBRARIES ${SDL2_LIBRARIES})

find_package(glfw3 REQUIRED)
list(APPEND IMGUI_LIBRARIES glfw)

set(OpenGL_GL_PREFERENCE LEGACY)
find_package(OpenGL REQUIRED)
list(APPEND IMGUI_LIBRARIES OpenGL::GL)

find_package(Vulkan REQUIRED)
list(APPEND IMGUI_LIBRARIES Vulkan::Vulkan)

set(IMGUI_STATIC "no" CACHE STRING "Build as a static library")
set(IMGUI_FREETYPE "no" CACHE STRING "Build with freetype library")

if(IMGUI_FREETYPE)
	find_package(freetype REQUIRED PATHS ${FREETYPE_PATH})
	list(APPEND IMGUI_LIBRARIES freetype)
	list(APPEND IMGUI_SOURCES imgui/misc/freetype/imgui_freetype.cpp)
	add_definitions("-DCIMGUI_FREETYPE=1")
endif(IMGUI_FREETYPE)

#add library and link
if (IMGUI_STATIC)
    add_library(cimgui STATIC ${IMGUI_SOURCES})
else (IMGUI_STATIC)
    add_library(cimgui SHARED ${IMGUI_SOURCES})
endif (IMGUI_STATIC)

target_compile_definitions(cimgui PUBLIC IMGUI_DISABLE_OBSOLETE_FUNCTIONS=1)
if (WIN32)
    target_compile_definitions(cimgui PUBLIC IMGUI_IMPL_API=extern\t\"C\"\t__declspec\(dllexport\))
    list(APPEND IMGUI_LIBRARIES imm32)
else (WIN32)
    target_compile_definitions(cimgui PUBLIC IMGUI_IMPL_API=extern\t\"C\"\t)
endif (WIN32)

target_include_directories(cimgui PUBLIC ${CMAKE_CURRENT_SOURCE_DIR})
target_include_directories(cimgui PUBLIC ${CMAKE_CURRENT_SOURCE_DIR}/imgui)
set_target_properties(cimgui PROPERTIES PREFIX "")
target_link_libraries(cimgui ${IMGUI_LIBRARIES})
target_include_directories(cimgui PUBLIC ${SDL2_INCLUDE_DIRS})
target_include_directories(cimgui PUBLIC ${glfw3_INCLUDE_DIR})

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
