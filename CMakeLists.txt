

cmake_minimum_required(VERSION 3.5...3.25)
project(Tetris LANGUAGES CXX)
set(PACKAGE_VERSION "1.0.0")
set(CPACK_PACKAGE_NAME "tetris-game")
set(CPACK_PACKAGE_VENDOR "ka-techkaa")
set(CPACK_PACKAGE_DESCRIPTION "Classic Tetris Game using SFML")
set(CPACK_PACKAGE_CONTACT "kate.cherkasina@gmail.com")
set(CPACK_DEBIAN_PACKAGE_MAINTAINER "ka-techkaa")
set(CPACK_DEBIAN_PACKAGE_DEPENDS "libsfml-dev (>= 2.5.1), libc6 (>= 2.27)")
set(CPACK_GENERATOR "DEB")
include(FetchContent)
include(CPack)
find_package(SFML REQUIRED network audio graphics window system)

if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/images)
    file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/images DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
else()
    message(WARNING "Папка images не найдена!")
endif()

add_executable(racing main.cpp)
target_compile_features(racing PRIVATE cxx_std_17)
target_link_libraries(racing PRIVATE sfml-system sfml-window sfml-graphics sfml-network sfml-audio)

find_package(SFML ${SFML_VERSION} QUIET COMPONENTS graphics window system audio network)

if(NOT SFML_FOUND)
    message(STATUS "SFML не найден, пробуем скачать...")
    set(SFML_URL_OPTIONS
        URL https://www.sfml-dev.org/files/SFML-${SFML_VERSION}-sources.zip
        URL https://github.com/SFML/SFML/archive/refs/tags/${SFML_VERSION}.zip
    )
    FetchContent_Declare(
        sfml
        ${SFML_URL_OPTIONS}
        SOURCE_DIR ${CMAKE_BINARY_DIR}/_deps/sfml-src
        BINARY_DIR ${CMAKE_BINARY_DIR}/_deps/sfml-build
        DOWNLOAD_EXTRACT_TIMESTAMP TRUE
    )
    set(BUILD_SHARED_LIBS OFF CACHE BOOL "Build static libraries")
    set(SFML_BUILD_DOC FALSE CACHE BOOL "Don't build documentation")
    set(SFML_BUILD_EXAMPLES FALSE CACHE BOOL "Don't build examples")
    FetchContent_MakeAvailable(sfml)
    set(SFML_ROOT ${CMAKE_BINARY_DIR}/_deps/sfml-build)
    set(SFML_INCLUDE_DIR ${CMAKE_BINARY_DIR}/_deps/sfml-src/include)
    find_package(SFML ${SFML_VERSION} REQUIRED COMPONENTS graphics window system audio network)
endif()

if(NOT SFML_FOUND)
    message(FATAL_ERROR "SFML не найден и не может быть загружен автоматически")
endif()

if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/images)
    file(COPY ${CMAKE_CURRENT_SOURCE_DIR}/images DESTINATION ${CMAKE_CURRENT_BINARY_DIR})
else()
    message(WARNING "Папка images не найдена!")
endif()

install(TARGETS racing 
    RUNTIME DESTINATION bin
    BUNDLE DESTINATION bin
)

if(EXISTS ${CMAKE_CURRENT_SOURCE_DIR}/images)
    install(DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/images
        DESTINATION share/${PROJECT_NAME}
    )
endif()

if(WIN32)
    add_custom_command(TARGET racing POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
        "$<TARGET_FILE:sfml-graphics>"
        "$<TARGET_FILE:sfml-window>"
        "$<TARGET_FILE:sfml-system>"
        "$<TARGET_FILE:sfml-audio>"
        "$<TARGET_FILE:sfml-network>"
        "$<TARGET_FILE_DIR:racing>"
    )
    install(FILES
        "$<TARGET_FILE:sfml-graphics>"
        "$<TARGET_FILE:sfml-window>"
        "$<TARGET_FILE:sfml-system>"
        "$<TARGET_FILE:sfml-audio>"
        "$<TARGET_FILE:sfml-network>"
        DESTINATION bin
    )
endif()
