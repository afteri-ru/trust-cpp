# Создадим цель для тестов
add_executable(memsafe_unittest
    _cycles.cpp
    _example.cpp
    memsafe_test.cpp
)

# Настроим тесты
set_target_properties(memsafe_unittest PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
)

# Используем общую функцию для установки свойств
set_common_target_properties(memsafe_unittest)

target_compile_definitions(memsafe_unittest PRIVATE
    BUILD_UNITTEST
)

target_include_directories(memsafe_unittest PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/..
    ${CMAKE_CURRENT_SOURCE_DIR}/../plugin
    ${YAML_CPP_INCLUDE_DIRS}
)

target_link_libraries(memsafe_unittest 
    GTest::GTest 
    GTest::Main 
    yaml-cpp
    pthread
)

# Убедимся, что плагин собран перед запуском тестов
add_dependencies(memsafe_unittest memsafe_clang)

# Добавим пользовательские команды для анализа файлов с использованием плагина
# Для _cycles.cpp
add_memsafe_analysis_command(
    ${CMAKE_CURRENT_SOURCE_DIR}/_cycles.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/_cycles
    memsafe_unittest
    -Xclang -plugin-arg-memsafe -Xclang circleref-read=${CMAKE_CURRENT_SOURCE_DIR}/circleref.memsafe
)

# Для _example.cpp
add_memsafe_analysis_command(
    ${CMAKE_CURRENT_SOURCE_DIR}/_example.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/_example
    memsafe_unittest
    -Xclang -plugin-arg-memsafe -Xclang circleref-disable
)

# Создадим цель для запуска тестов
add_custom_target(run_tests
    COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/memsafe_unittest
    DEPENDS memsafe_unittest
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    COMMENT "Running memsafe unit tests"
)
