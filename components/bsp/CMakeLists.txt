message(STATUS "PROJECT_DIR: " ${PROJECT_DIR})
set(bsp_src "src/storage/bsp_sdcard.c")

if(EXISTS ${PROJECT_DIR}/sdkconfig)
    file(READ ${PROJECT_DIR}/sdkconfig SDKCONFIG_RULE)
    string(REGEX MATCH "CONFIG_BSP_BOARD_ESP32_S3_BOX_Lite=y" CMAKE_TARGET_IS_BOX_LITE "${SDKCONFIG_RULE}")
    string(REGEX MATCH "CONFIG_BSP_BOARD_ESP32_S3_BOX=y" CMAKE_TARGET_IS_ESP_BOX "${SDKCONFIG_RULE}")
else()
    message(STATUS "sdkconfig not exist")
    file(READ ${CMAKE_CURRENT_LIST_DIR}/Kconfig.projbuild SDKCONFIG_RULE)
    string(REGEX MATCH "default BSP_BOARD_ESP32_S3_BOX_Lite" CMAKE_TARGET_IS_BOX_LITE "${SDKCONFIG_RULE}")
    string(REGEX MATCH "default BSP_BOARD_ESP32_S3_BOX" CMAKE_TARGET_IS_ESP_BOX "${SDKCONFIG_RULE}")
endif()

if (CMAKE_TARGET_IS_ESP_BOX)
    message(STATUS "PLATFORM ESP32_S3_BOX.")
    list(APPEND bsp_src "src/boards/esp32_s3_box.c")
elseif (CMAKE_TARGET_IS_BOX_LITE)
    message(STATUS "PLATFORM ESP32_S3_BOX_Lite.")
    list(APPEND bsp_src "src/boards/esp32_s3_box_lite.c")
    set(box_lite "-lite")
else()
    message(FATAL_ERROR "PLATFORM unknown.")
endif()

string(REGEX MATCH "factory_demo" PROJECT_IS_FACTORY_DEMO "${PROJECT_DIR}")
if (PROJECT_IS_FACTORY_DEMO)
set(priv_requires "esp-box${box_lite}" "aht20" "at581x")
else()
set(priv_requires "esp-box${box_lite}")
endif()

set(requires "driver" "fatfs")

idf_component_register(
    SRCS ${bsp_src}
    INCLUDE_DIRS
        "src/boards"
        "include"
    PRIV_REQUIRES ${priv_requires}
    REQUIRES ${requires})

if (PROJECT_IS_FACTORY_DEMO)
set_source_files_properties(
    SRCS ${bsp_src}
    PROPERTIES COMPILE_OPTIONS
    -DCONFIG_NEED_SENSOR)
endif()
