cmake_minimum_required (VERSION 3.28)

project (VulkanTutorial)

list(APPEND CMAKE_MODULE_PATH "${CMAKE_SOURCE_DIR}/CMake")

find_package (SDL3 CONFIG REQUIRED)
find_package (glm REQUIRED)
find_package (Vulkan REQUIRED)
find_package (tinyobjloader REQUIRED)
find_package(stb REQUIRED)

find_program (SLANGC_EXECUTABLE slangc REQUIRED)

function (add_shaders_target TARGET)
    cmake_parse_arguments (
        "SHADER"
        ""
        "CHAPTER_NAME"
        "SOURCES"
        ${ARGN}
    )

    list(GET SHADER_SOURCES 0 SHADER_SOURCE)

    set (SHADERS_DIR
        "${CMAKE_CURRENT_BINARY_DIR}/${SHADER_CHAPTER_NAME}/shaders"
    )

    set (SPV_FILE "${SHADERS_DIR}/slang.spv")

    add_custom_command (
        OUTPUT "${SPV_FILE}"
        COMMAND ${CMAKE_COMMAND} -E make_directory
            "${SHADERS_DIR}"
        COMMAND "${SLANGC_EXECUTABLE}"
            "${CMAKE_CURRENT_SOURCE_DIR}/${SHADER_SOURCE}"
            -entry vertMain
            -entry fragMain
            -target spirv
            -fvk-use-entrypoint-name
            -o "${SPV_FILE}"
        DEPENDS "${CMAKE_CURRENT_SOURCE_DIR}/${SHADER_SOURCE}"
        COMMENT "Compiling Slang Shader: ${SHADER_SOURCE}"
        VERBATIM
    )

    add_custom_target (
        ${TARGET}
        DEPENDS "${SPV_FILE}"
    )
endfunction ()

function (add_chapter CHAPTER_NAME)
  cmake_parse_arguments (CHAPTER "" "SHADER" "LIBS;TEXTURES;MODELS" ${ARGN})

  add_executable (${CHAPTER_NAME} ${CHAPTER_NAME}.cpp)
  set_target_properties (${CHAPTER_NAME} PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY ${CMAKE_BINARY_DIR}/${CHAPTER_NAME})
  set_target_properties (${CHAPTER_NAME} PROPERTIES CXX_STANDARD 17)
  target_link_libraries (${CHAPTER_NAME} Vulkan::Vulkan SDL3::SDL3 stb::stb)

if (DEFINED CHAPTER_SHADER)
    set (CHAPTER_SHADER_TARGET ${CHAPTER_NAME}_shader)
    set (SHADER_SOURCES "${CHAPTER_SHADER}.slang")
    add_shaders_target (
        ${CHAPTER_SHADER_TARGET}
        CHAPTER_NAME ${CHAPTER_NAME}
        SOURCES ${SHADER_SOURCES}
    )
    add_dependencies (
        ${CHAPTER_NAME}
        ${CHAPTER_SHADER_TARGET}
    )
    add_custom_command (
        TARGET ${CHAPTER_NAME} POST_BUILD
        COMMAND ${CMAKE_COMMAND} -E make_directory
            "$<TARGET_FILE_DIR:${CHAPTER_NAME}>/shaders"
        COMMAND ${CMAKE_COMMAND} -E copy_if_different
            "${CMAKE_CURRENT_BINARY_DIR}/${CHAPTER_NAME}/shaders/slang.spv"
            "$<TARGET_FILE_DIR:${CHAPTER_NAME}>/shaders/slang.spv"
        COMMENT "Copying Slang shader to executable directory"
        VERBATIM
    )
endif ()
endfunction ()

add_chapter (00_base_code)

add_chapter (01_instance_creation)

add_chapter (02_validation_layers)

add_chapter (03_physical_device_selection)

add_chapter (04_logical_device)

add_chapter (05_window_surface)

add_chapter (06_swap_chain_creation)

add_chapter (07_image_views)

add_chapter (08_graphics_pipeline)

add_chapter (09_shader_modules
  SHADER 09_shader_base)

add_chapter (10_fixed_functions
  SHADER 09_shader_base)

add_chapter (11_render_passes
  SHADER 09_shader_base)

add_chapter (12_graphics_pipeline_complete
  SHADER 09_shader_base)

add_chapter (13_framebuffers
  SHADER 09_shader_base)

add_chapter (14_command_buffers
  SHADER 09_shader_base)

add_chapter (15_hello_triangle
  SHADER 09_shader_base)
