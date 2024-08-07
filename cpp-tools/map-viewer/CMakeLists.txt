project(bird)

set(extra_dependencies "")
set(extra_libraries "")

find_package(FreeGLUT)
if (FreeGLUT_FOUND)
	set(extra_dependencies ${extra_dependencies}
		FreeGLUT::freeglut
	)
	set(extra_libraries ${extra_libraries}
		FreeGLUT::freeglut
	)
else()
	find_library(FREEGLUT_LIBRARY freeglut)
	if (FREEGLUT_LIBRARY STREQUAL "FREEGLUT_LIBRARY-NOTFOUND")
		add_subdirectory("${OSMAND_ROOT}/tools/cpp-tools/map-viewer/externals/freeglut" "externals/freeglut")
		set(extra_dependencies ${extra_dependencies}
			freeglut_static
		)
		set(extra_libraries ${extra_libraries}
			freeglut_static
		)
	else()
		set(extra_libraries ${extra_libraries}
			${FREEGLUT_LIBRARY}
		)
	endif()
endif()

set(target_specific_sources "")
set(target_specific_private_definitions "")
if (CMAKE_TARGET_OS STREQUAL "macosx")
	set(target_specific_sources ${target_specific_sources}
		"x11.cpp"
	)
	set(target_specific_private_definitions ${target_specific_private_definitions}
		-DGL_SILENCE_DEPRECATION
	)
endif()

set(sources
	"main.cpp"
	${target_specific_sources}
)

if (CMAKE_SHARED_LIBS_ALLOWED_ON_TARGET)
	add_executable(bird
		${sources}
	)

	add_dependencies(bird
		OsmAndCore_shared
		OsmAndCore_ResourcesBundle_shared
		${extra_dependencies}
	)
	target_link_libraries(bird
		OsmAndCore_shared
		OsmAndCore_ResourcesBundle_shared
		${extra_libraries}
	)
	target_compile_definitions(bird
		PRIVATE
			${target_specific_private_definitions}
	)
endif()

if (TARGET OsmAndCore_static_standalone)
	add_executable(bird_standalone
		${sources}
	)

	add_dependencies(bird_standalone
		OsmAndCore_static_standalone
		OsmAndCore_ResourcesBundle_static
		${extra_dependencies}
	)
	target_link_libraries(bird_standalone
		OsmAndCore_static_standalone
		${extra_libraries}
	)
	target_compile_definitions(bird
		PRIVATE
			${target_specific_private_definitions}
	)
	link_entire_static_library(bird_standalone OsmAndCore_ResourcesBundle_static)
endif()
