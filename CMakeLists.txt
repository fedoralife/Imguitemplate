cmake_minimum_required(VERSION 3.10)
project(app VERSION 1.0)

# Set C++ standards
set(CMAKE_CXX_STANDARD 11)
set(CMAKE_CXX_STANDARD_REQUIRED True)

# Ensure the compile_commands.json file is generated
set(CMAKE_EXPORT_COMPILE_COMMANDS ON)

# Set output directory for executables
set(CMAKE_RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/bin)

# Check platform and set up accordingly
if (WIN32)
    message(STATUS "Configuring for Windows...")

    # For MSYS2/MINGW, check if we have the proper compilers
    find_program(GCC_BIN g++)
    find_program(CLANG_BIN clang)

    if (GCC_BIN)
        set(CMAKE_CXX_COMPILER ${GCC_BIN})
    elseif (CLANG_BIN)
        set(CMAKE_CXX_COMPILER ${CLANG_BIN})
    else()
        message(FATAL_ERROR "Neither GCC nor Clang found! Please install a compiler.")
    endif()

    # Link necessary libraries for Windows
    set(LIBS -lglfw3 -lgdi32 -lopengl32 -limm32)

elseif (APPLE)
    message(STATUS "Configuring for macOS...")

    # Use clang on macOS
    set(CMAKE_CXX_COMPILER clang++)

    # Link macOS specific libraries
    set(LIBS -framework OpenGL -framework Cocoa -framework IOKit -framework CoreVideo)
    find_package(OpenGL REQUIRED)

elseif (UNIX)
    message(STATUS "Configuring for Linux...")

    # Use gcc or clang on Linux
    find_program(GCC_BIN g++)
    find_program(CLANG_BIN clang)

    if (GCC_BIN)
        set(CMAKE_CXX_COMPILER ${GCC_BIN})
    elseif (CLANG_BIN)
        set(CMAKE_CXX_COMPILER ${CLANG_BIN})
    else()
        message(FATAL_ERROR "Neither GCC nor Clang found! Please install a compiler.")
    endif()

    # Find OpenGL and GLFW
    find_package(OpenGL REQUIRED)
    find_package(glfw3 REQUIRED)

    # Link OpenGL and GLFW libraries
    set(LIBS OpenGL::GL glfw)

endif()

# Add the source files (adjust this pattern to match your source files location)
file(GLOB SOURCES "src/*.cpp")

# Add ImGui source files
set(IMGUI_DIR "./vendor")
file(GLOB IMGUI_SOURCES
    ${IMGUI_DIR}/imgui.cpp
    ${IMGUI_DIR}/imgui_demo.cpp
    ${IMGUI_DIR}/imgui_draw.cpp
    ${IMGUI_DIR}/imgui_tables.cpp
    ${IMGUI_DIR}/imgui_widgets.cpp
    ${IMGUI_DIR}/backends/imgui_impl_glfw.cpp
    ${IMGUI_DIR}/backends/imgui_impl_opengl3.cpp
)

# Create executable target
add_executable(${PROJECT_NAME} ${SOURCES} ${IMGUI_SOURCES})

# Include directories for ImGui and your custom headers
target_include_directories(${PROJECT_NAME} PRIVATE ${IMGUI_DIR} ${IMGUI_DIR}/backends)

# Add header directory (adjust if needed)
target_include_directories(${PROJECT_NAME} PRIVATE ${CMAKE_SOURCE_DIR}/header)

# Link libraries (OpenGL, GLFW, etc.)
target_link_libraries(${PROJECT_NAME} PRIVATE ${LIBS})

# Compiler options
if (MSVC)
    target_compile_options(${PROJECT_NAME} PRIVATE /W4 /permissive-)
else()
    target_compile_options(${PROJECT_NAME} PRIVATE -Wall -Wextra -Wpedantic)
endif()

# Install rules (optional, modify if needed)
install(TARGETS ${PROJECT_NAME} DESTINATION bin)

