cmake_minimum_required(VERSION 2.8)

project(hlsl2glslfork)

set(CMAKE_SUPPRESS_REGENERATION TRUE)

set(HEADER_FILES
  hlslang/Include/BaseTypes.h
  hlslang/Include/Common.h
  hlslang/Include/InfoSink.h
  hlslang/Include/InitializeGlobals.h
  hlslang/Include/InitializeParseContext.h
  hlslang/Include/intermediate.h
  hlslang/Include/PoolAlloc.h
  hlslang/Include/Types.h
  hlslang/MachineIndependent/SymbolTable.h
)
source_group("Header Files" FILES ${HEADER_FILES})


set(GLSL_CODE_GEN_FILES 
  hlslang/GLSLCodeGen/glslCommon.cpp
  hlslang/GLSLCodeGen/glslCommon.h
  hlslang/GLSLCodeGen/glslFunction.cpp
  hlslang/GLSLCodeGen/glslFunction.h
  hlslang/GLSLCodeGen/glslOutput.cpp
  hlslang/GLSLCodeGen/glslOutput.h
  hlslang/GLSLCodeGen/glslStruct.cpp
  hlslang/GLSLCodeGen/glslStruct.h
  hlslang/GLSLCodeGen/glslSymbol.cpp
  hlslang/GLSLCodeGen/glslSymbol.h
  hlslang/GLSLCodeGen/hlslCrossCompiler.cpp
  hlslang/GLSLCodeGen/hlslCrossCompiler.h
  hlslang/GLSLCodeGen/hlslLinker.cpp
  hlslang/GLSLCodeGen/hlslLinker.h
  hlslang/GLSLCodeGen/hlslSupportLib.cpp
  hlslang/GLSLCodeGen/hlslSupportLib.h
  hlslang/GLSLCodeGen/propagateMutable.cpp
  hlslang/GLSLCodeGen/propagateMutable.h
  hlslang/GLSLCodeGen/typeSamplers.cpp
  hlslang/GLSLCodeGen/typeSamplers.h
)
source_group("GLSL Code Gen" FILES ${GLSL_CODE_GEN_FILES})

set(INCLUDE_FILES
  include
)

source_group("include" FILES ${INCLUDE_FILES})

set(MACHINE_INDEPENDENT_FILES
  hlslang/MachineIndependent/HLSL2GLSL.cpp
  hlslang/MachineIndependent/hlslang.l
  hlslang/MachineIndependent/hlslang.y
  hlslang/MachineIndependent/InfoSink.cpp
  hlslang/MachineIndependent/Initialize.cpp
  hlslang/MachineIndependent/Initialize.h
  hlslang/MachineIndependent/Intermediate.cpp
  hlslang/MachineIndependent/intermOut.cpp
  hlslang/MachineIndependent/IntermTraverse.cpp
  hlslang/MachineIndependent/localintermediate.h
  #hlslang/MachineIndependent/parseConst.cpp
  hlslang/MachineIndependent/ParseHelper.cpp
  hlslang/MachineIndependent/ParseHelper.h
  hlslang/MachineIndependent/PoolAlloc.cpp
  hlslang/MachineIndependent/RemoveTree.cpp
  hlslang/MachineIndependent/RemoveTree.h
  hlslang/MachineIndependent/SymbolTable.cpp
  hlslang/MachineIndependent/SymbolTable.h
  hlslang/MachineIndependent/ConstantFolding.cpp
)

source_group("Machine Independent" FILES ${MACHINE_INDEPENDENT_FILES})

set(MACHINE_INDEPENDENT_CPP_FILES
  hlslang/MachineIndependent/preprocessor/mojoshader_common.cpp
  hlslang/MachineIndependent/preprocessor/mojoshader_internal.h
  hlslang/MachineIndependent/preprocessor/mojoshader_lexer.cpp
  hlslang/MachineIndependent/preprocessor/mojoshader_preprocessor.cpp
  hlslang/MachineIndependent/preprocessor/mojoshader.cpp
  hlslang/MachineIndependent/preprocessor/mojoshader.h
  hlslang/MachineIndependent/preprocessor/sourceloc.cpp
  hlslang/MachineIndependent/preprocessor/sourceloc.h
)

source_group("Machine Independent\\CPP" FILES ${MACHINE_INDEPENDENT_CPP_FILES})


if (WIN32)
  EXEC_PROGRAM(${CMAKE_COMMAND} ARGS "-E write_regv \"${hkey}\" \"${dir}\"")
endif ()

if (WIN32 OR APPLE)
	set(MACHINE_INDEPENDENT_GENERATED_SOURCE_FILES
    ${CMAKE_CURRENT_SOURCE_DIR}/hlslang/MachineIndependent/Gen_hlslang.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/hlslang/MachineIndependent/Gen_hlslang_tab.cpp
	  ${CMAKE_CURRENT_SOURCE_DIR}/hlslang/MachineIndependent/hlslang_tab.h
	)
else()
	set(MACHINE_INDEPENDENT_GENERATED_SOURCE_FILES
	  ${CMAKE_CURRENT_SOURCE_DIR}/hlslang/MachineIndependent/hlslang_tab.cpp
	  ${CMAKE_CURRENT_SOURCE_DIR}/hlslang/MachineIndependent/hlslang_tab.h
	)
endif()

SET_SOURCE_FILES_PROPERTIES(${MACHINE_INDEPENDENT_GENERATED_SOURCE_FILES} PROPERTIES
  GENERATED 1
)

source_group("Machine Independent\\Generated Source" FILES ${MACHINE_INDEPENDENT_GENERATED_SOURCE_FILES})
                                         

# Add system specific settings
if (WIN32)
    set(OSDEPENDENT_FILES
      hlslang/OSDependent/Windows/unistd.h
      hlslang/OSDependent/Windows/osinclude.h
      hlslang/OSDependent/Windows/ossource.cpp
    )
    source_group("OSDependent\\Windows" FILES ${OSDEPENDENT_FILES})
    include_directories(${CMAKE_CURRENT_SOURCE_DIR}/hlslang/OSDependent/Windows)
    
    add_custom_command(OUTPUT hlslang/MachineIndependent/Gen_hlslang_tab.cpp hlslang/MachineIndependent/hlslang.tab.h 
                         COMMAND set ARGS "PATH=%PATH%;${CMAKE_CURRENT_SOURCE_DIR}/tools/bin/"
                         COMMAND set ARGS "BISON_SIMPLE=${CMAKE_CURRENT_SOURCE_DIR}/tools/bin/bison.simple"
                         COMMAND set ARGS "BISON_HAIRY=${CMAKE_CURRENT_SOURCE_DIR}/tools/bin/bison.simple"
                         COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/tools/bin/bison.exe ARGS  -d -t -v hlslang.y
                         COMMAND copy ARGS /y hlslang.tab.c Gen_hlslang_tab.cpp
                         COMMAND del ARGS hlslang.tab.c
                         COMMAND copy ARGS /y hlslang.tab.h hlslang_tab.h
                         COMMAND del ARGS hlslang.tab.h
                         COMMAND del ARGS hlslang.output
                         MAIN_DEPENDENCY hlslang/MachineIndependent/hlslang.y
                         WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/hlslang/MachineIndependent
                         COMMENT "Executing Bison on hlslang.y"
                      )
                   
    add_custom_command(OUTPUT hlslang/MachineIndependent/Gen_hlslang.cpp
                         COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/tools/flex.exe ARGS hlslang.l
                         MAIN_DEPENDENCY hlslang/MachineIndependent/hlslang.l
                         DEPENDS hlslang/MachineIndependent/hlslang_tab.h
                         WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/hlslang/MachineIndependent
                         COMMENT "Executing flex on hlslang.l"
                      )
    
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} /D\"_HAS_ITERATOR_DEBUGGING=0\" /D\"_SECURE_SCL=0\" /D\"_CRT_SECURE_NO_WARNINGS\"")
    SET(TEST_LIBS opengl32.lib)
elseif (APPLE)
    set(OSDEPENDENT_FILES
      hlslang/OSDependent/Mac/osinclude.h
      hlslang/OSDependent/Mac/ossource.cpp
    )
    source_group("OSDependent\\Mac" FILES ${OSDEPENDENT_FILES})
    include_directories(${CMAKE_CURRENT_SOURCE_DIR}/hlslang/OSDependent/Mac)    

    add_custom_command(OUTPUT hlslang/MachineIndependent/Gen_hlslang_tab.cpp hlslang/MachineIndependent/hlslang_tab.h 
                         COMMAND set ARGS "BISON_SIMPLE=../../tools/bison.simple"
                         COMMAND set ARGS "BISON_HAIRY=../../tools/bison.simple"
                         COMMAND bison ARGS -o hlslang_tab.cpp -d -t -v hlslang.y
                         COMMAND mv hlslang_tab.hpp hlslang_tab.h
                         MAIN_DEPENDENCY hlslang/MachineIndependent/hlslang.y
                         WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/hlslang/MachineIndependent
                         COMMENT "Executing Bison on hlslang.y"
                      )
                   
    add_custom_command(OUTPUT hlslang/MachineIndependent/Gen_hlslang.cpp
                         COMMAND flex ARGS hlslang.l
                         MAIN_DEPENDENCY hlslang/MachineIndependent/hlslang.l
                         DEPENDS hlslang/MachineIndependent/hlslang_tab.h
                         WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/hlslang/MachineIndependent
                         COMMENT "Executing flex on hlslang.l"
                      )
    FIND_LIBRARY(OPENGL_LIBRARY OpenGL)
	FIND_LIBRARY(COCOA_LIBRARY Cocoa)
    SET(TEST_LIBS ${OPENGL_LIBRARY} ${COCOA_LIBRARY})
elseif (UNIX)
    set(OSDEPENDENT_FILES
      hlslang/OSDependent/Linux/osinclude.h
      hlslang/OSDependent/Linux/ossource.cpp
    )
    source_group("OSDependent\\Linux" FILES ${OSDEPENDENT_FILES})
    include_directories(${CMAKE_CURRENT_SOURCE_DIR}/hlslang/OSDependent/Linux)

    add_custom_command(OUTPUT hlslang/MachineIndependent/hlslang_tab.cpp hlslang/MachineIndependent/hlslang_tab.h 
                         COMMAND set ARGS "BISON_SIMPLE=../../tools/bison.simple"
                         COMMAND set ARGS "BISON_HAIRY=../../tools/bison.simple"
                         COMMAND bison ARGS -o hlslang_tab.cpp -d -t -v hlslang.y
                         COMMAND mv hlslang_tab.hpp hlslang_tab.h
                         MAIN_DEPENDENCY hlslang/MachineIndependent/hlslang.y
                         WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/hlslang/MachineIndependent
                         COMMENT "Executing Bison on hlslang.y"
                      )
                   
    add_custom_command(OUTPUT hlslang/MachineIndependent/Gen_hlslang.cpp
                         COMMAND flex ARGS hlslang.l
                         MAIN_DEPENDENCY hlslang/MachineIndependent/hlslang.l
                         DEPENDS hlslang/MachineIndependent/hlslang_tab.h
                         WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/hlslang/MachineIndependent
                         COMMENT "Executing flex on hlslang.l"
                      )
    SET(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -ggdb")
    SET(TEST_LIBS GL glut GLEW pthread)
endif ()


include_directories(
  ${CMAKE_CURRENT_SOURCE_DIR}/hlslang
  ${CMAKE_CURRENT_SOURCE_DIR}/hlslang/MachineIndependent
)

add_library(hlsl2glsl 
                ${HEADER_FILES} 
                ${GLSL_CODE_GEN_FILES} 
                ${INCLUDE_FILES} 
                ${MACHINE_INDEPENDENT_FILES} 
                ${MACHINE_INDEPENDENT_CPP_FILES} 
                ${OSDEPENDENT_FILES}
                ${MACHINE_INDEPENDENT_GENERATED_SOURCE_FILES}
           )


include_directories(
    ${INCLUDE_FILES} 
)
add_executable(hlsl2glsltest tests/hlsl2glsltest/hlsl2glsltest.cpp)

target_link_libraries(hlsl2glsltest hlsl2glsl ${TEST_LIBS})
