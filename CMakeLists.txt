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

if(NOT MSVC)
    target_compile_options(yapb PRIVATE
        -fno-threadsafe-statics
        -pthread
    )
endif()

if(MSVC)
    target_link_libraries(yapb PRIVATE ws2_32)
elseif(ANDROID)
    target_compile_definitions(yapb PRIVATE -DCR_ANDROID)
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
		DESTINATION "${GAME_DIR}/${CLIENT_INSTALL_DIR}/" OPTIONAL)
endif()