if(PLATFORM_WIN32 OR PLATFORM_LINUX)
    cmake_minimum_required (VERSION 3.13)
else()
    cmake_minimum_required (VERSION 3.6)
endif()


project(forwardplusrendering CXX)

#set(SOURCE src/Tutorial02_Cube.cpp)
#set(INCLUDE src/Tutorial02_Cube.h)

MACRO(ADD_MSVC_PRECOMPILED_HEADER PrecompiledHeader PrecompiledSource SourcesVar)
  IF(MSVC)
    GET_FILENAME_COMPONENT(PrecompiledBasename ${PrecompiledHeader} NAME_WE)
    SET(PrecompiledBinary "${CMAKE_CURRENT_BINARY_DIR}/${PrecompiledBasename}.pch")
    SET(Sources ${${SourcesVar}})

    SET_SOURCE_FILES_PROPERTIES(${PrecompiledSource}
                                PROPERTIES COMPILE_FLAGS "/Yc \"${PrecompiledHeader}\" /Fp\"${PrecompiledBinary}\""
                                           OBJECT_OUTPUTS "${PrecompiledBinary}")
    SET_SOURCE_FILES_PROPERTIES(${Sources}
                                PROPERTIES COMPILE_FLAGS "/Yu \"${PrecompiledHeader}\" /FI \"${PrecompiledHeader}\" /Fp \"${PrecompiledBinary}\""
                                           OBJECT_DEPENDS "${PrecompiledBinary}")  
    # Add precompiled header to SourcesVar
    LIST(APPEND ${SourcesVar} ${PrecompiledSource})
  ENDIF(MSVC)
ENDMACRO(ADD_MSVC_PRECOMPILED_HEADER)

function(add_win32_app TARGET_NAME SOURCE INCLUDE ASSETS)
	add_executable(${TARGET_NAME} WIN32 ${SOURCE} ${INCLUDE} ${ASSETS})

	if (CMAKE_CXX_COMPILER_ID MATCHES "GNU")
		# libmingw32 must be included BEFORE NativeAppBase that contains the definition of WinMain.
		# otherwise WinMain will be stripped out of NativeAppBase and will be unresolved.
		target_link_libraries(${TARGET_NAME}
		PRIVATE
			mingw32
		)
	endif()
endfunction()

function(add_target_platform_app TARGET_NAME SOURCE INCLUDE ASSETS)
	add_win32_app("${TARGET_NAME}" "${SOURCE}" "${INCLUDE}" "${ASSETS}")
endfunction()


add_subdirectory(thirdparty/glm-0.9.9.6)
add_subdirectory(thirdparty/assimp-5.0.0)

#add_subdirectory(thirdparty/zlib)
add_subdirectory(thirdparty/freeimage)

set(ASSETS "")

file(GLOB_RECURSE SRC_SOURCE
	RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}
	"")
	
file(GLOB_RECURSE ENGINE_SOURCE 
	RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}
	${CMAKE_CURRENT_SOURCE_DIR}/engine/src/*.*
	${CMAKE_CURRENT_SOURCE_DIR}/engine/inc/*.*)

file(GLOB_RECURSE TEST_SOURCE 
	RELATIVE ${CMAKE_CURRENT_SOURCE_DIR}
	${CMAKE_CURRENT_SOURCE_DIR}/test/src/*.*
	${CMAKE_CURRENT_SOURCE_DIR}/test/inc/*.*)

# ADD_MSVC_PRECOMPILED_HEADER("${CMAKE_CURRENT_SOURCE_DIR}/engine/inc/EnginePCH.h" "${CMAKE_CURRENT_SOURCE_DIR}/engine/src/EnginePCH.cpp" ENGINE_SOURCE)

source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} FILES ${SRC_SOURCE})
source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} FILES ${ENGINE_SOURCE})
source_group(TREE ${CMAKE_CURRENT_SOURCE_DIR} FILES ${TEST_SOURCE})

set(SOURCE ${SOURCE} ${SRC_SOURCE})
set(SOURCE ${SOURCE} ${ENGINE_SOURCE})
set(SOURCE ${SOURCE} ${TEST_SOURCE})

set(PROJECT_NAME_TMP "forwardplusrendering")

add_target_platform_app(${PROJECT_NAME_TMP} "${SOURCE}" "${INCLUDE}" "${ASSETS}")

set_target_properties(${PROJECT_NAME_TMP} PROPERTIES 
		VS_DEBUGGER_WORKING_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}"
	)

set_target_properties(${PROJECT_NAME_TMP} PROPERTIES 
		VS_DEBUGGER_COMMAND_ARGUMENTS "-c ./conf/crytek-sponza.3dgep"
	)

target_link_libraries(${PROJECT_NAME_TMP} PRIVATE 
			AntTweakBar64
			freeimage mng openjpeg openexr rawlite tiff jpeg png
			assimp
			)

set (DE_TOP_PATH "${CMAKE_CURRENT_SOURCE_DIR}")

target_link_directories(${PROJECT_NAME_TMP} PRIVATE ${DE_TOP_PATH}/thirdparty/boost_1_70_0/stage/lib
			${DE_TOP_PATH}/thirdparty/AntTweakBar-1.16/lib
			)

#target_compile_definitions(${PROJECT_NAME_TMP} PRIVATE -DTW_STATIC)
#remove_definitions(-DTW_STATIC)

target_include_directories(${PROJECT_NAME_TMP} PRIVATE engine/inc/ 
						test/inc/
						${DE_TOP_PATH}/thirdparty/
						${DE_TOP_PATH}/thirdparty/glm-0.9.9.6/
						${DE_TOP_PATH}/thirdparty/assimp-5.0.0/include
						${CMAKE_BINARY_DIR}/thirdparty/assimp-5.0.0/include # assimp generated config.h			
						${DE_TOP_PATH}/thirdparty/boost_1_70_0
						${DE_TOP_PATH}/thirdparty/freeimage
						${DE_TOP_PATH}/thirdparty/AntTweakBar-1.16/include
						)

# copy dll
add_custom_command(TARGET ${PROJECT_NAME_TMP} POST_BUILD
                COMMAND ${CMAKE_COMMAND} -E copy_if_different
					"\"${DE_TOP_PATH}/thirdparty/AntTweakBar-1.16/lib/AntTweakBar64.dll\""
                    "\"$<TARGET_FILE_DIR:${PROJECT_NAME_TMP}>\""
					)
					
