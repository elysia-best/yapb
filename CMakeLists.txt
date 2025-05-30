cmake_minimum_required(VERSION 3.5)
project(yapb CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

set(YAPB_SRC
    "src/analyze.cpp"
    "src/botlib.cpp"
    "src/chatlib.cpp"
    "src/combat.cpp"
    "src/config.cpp"
    "src/control.cpp"
    "src/engine.cpp"
    "src/graph.cpp"
    "src/linkage.cpp"
    "src/manager.cpp"
    "src/module.cpp"
    "src/message.cpp"
    "src/navigate.cpp"
    "src/planner.cpp"
    "src/practice.cpp"
    "src/sounds.cpp"
    "src/storage.cpp"
    "src/support.cpp"
    "src/tasks.cpp"
    "src/vision.cpp"
    "src/vistable.cpp"
)

add_library(yapb MODULE ${YAPB_SRC})

if(CMAKE_CXX_COMPILER_ID MATCHES "Clang" OR CMAKE_CXX_COMPILER_ID MATCHES "GNU")
    target_compile_options(yapb PRIVATE -fno-threadsafe-statics -pthread)

    if(CMAKE_SYSTEM_PROCESSOR MATCHES "aarch64")
        target_compile_options(yapb PRIVATE -march=armv8-a+fp+simd)
    elseif(NOT CMAKE_SYSTEM_PROCESSOR MATCHES "arm" AND NOT CMAKE_SYSTEM_PROCESSOR MATCHES "^ppc" AND NOT CMAKE_SYSTEM_PROCESSOR MATCHES "^loong")
        target_compile_options(yapb PRIVATE -mmmx -msse -msse2 -msse3 -mssse3 -mfpmath=sse)
    endif()

    if(NOT CMAKE_BUILD_TYPE MATCHES "Debug")
        target_compile_options(yapb PRIVATE -funroll-loops -fomit-frame-pointer -fno-stack-protector -fvisibility=hidden -fvisibility-inlines-hidden)
    
        if(NOT WIN32 AND NOT CMAKE_SYSTEM_PROCESSOR MATCHES "aarch64" AND NOT CMAKE_SYSTEM_PROCESSOR STREQUAL "arm" AND NOT CMAKE_SYSTEM_PROCESSOR MATCHES "^ppc")
            target_compile_options(yapb PRIVATE -fdata-sections -ffunction-sections -fcf-protection=none)
            target_link_options(yapb PRIVATE -Wl,--version-script=${CMAKE_CURRENT_SOURCE_DIR}/ext/ldscripts/version.lds -Wl,--gc-sections)
    
            if(CMAKE_CXX_COMPILER_ID MATCHES "GNU")
                target_compile_options(yapb PRIVATE -fgraphite-identity -floop-nest-optimize)
                target_link_options(yapb PRIVATE -fgraphite-identity -floop-nest-optimize)
            endif()
        endif()

        if(NOT WIN32 AND CMAKE_CXX_COMPILER_ID MATCHES "GNU")
            target_link_options(yapb PRIVATE -flto-partition=none)
        endif()
    else()
        target_compile_options(yapb PRIVATE -g3 -ggdb -DDEBUG -D_FORTIFY_SOURCE=2)
    endif()

    if(WIN32 AND CMAKE_CXX_COMPILER_ID MATCHES "GNU")
        target_link_options(yapb PRIVATE -Xlinker --script -Xlinker ${CMAKE_CURRENT_SOURCE_DIR}/ext/ldscripts/i386pe.lds)
    endif()

    if(WIN32 AND NOT CMAKE_CXX_COMPILER_ID MATCHES "Clang")
        target_link_options(yapb PRIVATE -static-libgcc)
    endif()

    if(CMAKE_SIZEOF_VOID_P EQUAL 8 OR CMAKE_SYSTEM_PROCESSOR MATCHES "aarch64" OR CMAKE_SYSTEM_PROCESSOR STREQUAL "arm" OR CMAKE_SYSTEM_PROCESSOR MATCHES "^ppc")
        target_compile_options(yapb PRIVATE -fPIC)
        target_link_options(yapb PRIVATE -fPIC)
    endif()
elseif(WIN32 AND MSVC)
    if(NOT CMAKE_BUILD_TYPE MATCHES "Debug")
        target_compile_options(yapb PRIVATE /Zc:threadSafeInit- /GS- /Ob2 /Oy /Oi /Ot /fp:precise /GF /Gw /arch:SSE2 /Zi /guard:ehcont- /guard:cf- /GL /DEBUG)
        target_link_options(yapb PRIVATE /OPT:REF,ICF /GUARD:NO  /LTCG delayimp.lib /DELAYLOAD:user32.dll /DELAYLOAD:ws2_32.dll)
    endif()
endif()

if(WIN32)
    target_link_libraries(yapb PRIVATE user32 ws2_32)
elseif(ANDROID)
    target_link_libraries(yapb PRIVATE m dl log)
else()
    target_link_libraries(yapb PRIVATE m dl pthread)
endif()

target_include_directories(yapb PRIVATE
	${PROJECT_SRC_DIR}
	"inc"
	"ext"
    "ext/crlib"
    "ext/linkage"
)

install(TARGETS yapb
		DESTINATION "${GAME_DIR}/${SERVER_INSTALL_DIR}/"
		PERMISSIONS OWNER_READ OWNER_WRITE OWNER_EXECUTE
		GROUP_READ GROUP_EXECUTE
		WORLD_READ WORLD_EXECUTE)

# Install PDB file on Windows
if(MSVC)
	install(FILES $<TARGET_PDB_FILE:yapb>
		DESTINATION "${GAME_DIR}/${SERVER_INSTALL_DIR}/" OPTIONAL)
endif()
