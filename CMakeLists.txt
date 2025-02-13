cmake_minimum_required(VERSION 3.16)
set(CMAKE_SUPPRESS_REGENERATION true)

project(PotreeConverter LANGUAGES CXX)

set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED True)
set(CMAKE_BUILD_TYPE "Release" CACHE STRING "" FORCE)

#message(${PROJECT_SOURCE_DIR})

set(HEADER_FILES  
	./Converter/include/Attributes.h
	./Converter/include/chunker_countsort_laszip.h
	./Converter/include/ChunkRefiner.h
	./Converter/include/ConcurrentWriter.h
	./Converter/include/converter_utils.h
	./Converter/include/indexer.h
	./Converter/include/prototyping.h
	./Converter/include/sampler_poisson.h
	./Converter/include/sampler_poisson_average.h
	./Converter/include/sampler_random.h
	./Converter/include/structures.h
	./Converter/include/Vector3.h
	./Converter/include/PotreeConverter.h
	./Converter/include/logger.h
	./Converter/modules/LasLoader/LasLoader.h
	./Converter/modules/unsuck/unsuck.hpp
)

add_executable(PotreeConverter 
	./Converter/src/chunker_countsort_laszip.cpp
	./Converter/src/indexer.cpp 
	./Converter/src/main.cpp
	./Converter/src/logger.cpp
	./Converter/modules/LasLoader/LasLoader.cpp
	./Converter/modules/unsuck/unsuck_platform_specific.cpp
	${HEADER_FILES}
)

set(LASZIP_DIR "${PROJECT_SOURCE_DIR}/Converter/libs/laszip")
add_subdirectory(${LASZIP_DIR})
target_link_libraries(PotreeConverter laszip)

set(BROTLI_DIR "${PROJECT_SOURCE_DIR}/Converter/libs/brotli")
add_subdirectory(${BROTLI_DIR})
target_link_libraries(PotreeConverter brotlienc-static)
target_link_libraries(PotreeConverter brotlidec-static)


target_include_directories(PotreeConverter PRIVATE "./Converter/include")
target_include_directories(PotreeConverter PRIVATE "./Converter/modules")
target_include_directories(PotreeConverter PRIVATE "./Converter/libs")


if (UNIX)
	find_package(Threads REQUIRED)
	find_package(TBB REQUIRED)
	
	target_link_libraries(${PROJECT_NAME} Threads::Threads)
	target_link_libraries(${PROJECT_NAME} tbb)
	
	install(TARGETS PotreeConverter DESTINATION /usr/bin)
        install(TARGETS laszip DESTINATION /usr/lib)

	#SET(CMAKE_CXX_FLAGS "-pthread -ltbb")
endif (UNIX)

###############################################
# COPY PAGE TEMPLATE TO BINARY DIRECTORY
###############################################

add_custom_command(
	TARGET PotreeConverter POST_BUILD
	COMMAND ${CMAKE_COMMAND} -E copy_directory
		${PROJECT_SOURCE_DIR}/resources/page_template
		$<TARGET_FILE_DIR:${PROJECT_NAME}>/resources/page_template)

###############################################
# COPY LICENSE FILES TO BINARY DIRECTORY
###############################################
add_custom_command(
	TARGET PotreeConverter POST_BUILD
	COMMAND ${CMAKE_COMMAND} -E copy
		${LASZIP_DIR}/COPYING
		$<TARGET_FILE_DIR:${PROJECT_NAME}>/licenses/license_laszip.txt)

add_custom_command(
	TARGET PotreeConverter POST_BUILD
	COMMAND ${CMAKE_COMMAND} -E copy
		${BROTLI_DIR}/LICENSE
		$<TARGET_FILE_DIR:${PROJECT_NAME}>/licenses/license_brotli.txt)

add_custom_command(
	TARGET PotreeConverter POST_BUILD
	COMMAND ${CMAKE_COMMAND} -E copy
		${PROJECT_SOURCE_DIR}/Converter/libs/json/LICENSE
		$<TARGET_FILE_DIR:${PROJECT_NAME}>/licenses/license_json.txt)

add_custom_command(
	TARGET PotreeConverter POST_BUILD
	COMMAND ${CMAKE_COMMAND} -E copy
		${PROJECT_SOURCE_DIR}/LICENSE
		$<TARGET_FILE_DIR:${PROJECT_NAME}>/licenses/license_potree_converter.txt)

add_custom_command(
	TARGET PotreeConverter POST_BUILD
	COMMAND ${CMAKE_COMMAND} -E copy
		${PROJECT_SOURCE_DIR}/README.md
		$<TARGET_FILE_DIR:${PROJECT_NAME}>/README.md)

