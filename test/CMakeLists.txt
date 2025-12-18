# Создадим цель для тестов
add_executable(trusted-cpp_unittest
    _cycles.cpp
    _example.cpp
    trusted-cpp_test.cpp
)

# Настроим тесты
set_target_properties(trusted-cpp_unittest PROPERTIES
    RUNTIME_OUTPUT_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
)

# Используем общую функцию для установки свойств
set_common_target_properties(trusted-cpp_unittest)

target_compile_definitions(trusted-cpp_unittest PRIVATE
    BUILD_UNITTEST
)

target_include_directories(trusted-cpp_unittest PRIVATE
    ${CMAKE_CURRENT_SOURCE_DIR}/..
    ${CMAKE_CURRENT_SOURCE_DIR}/../plugin
    ${YAML_CPP_INCLUDE_DIRS}
)

target_link_libraries(trusted-cpp_unittest 
    GTest::GTest 
    GTest::Main 
    yaml-cpp
    pthread
)

# Убедимся, что плагин собран перед запуском тестов
add_dependencies(trusted-cpp_unittest trusted-cpp_clang)

# Добавим пользовательские команды для анализа файлов с использованием плагина
# Для _cycles.cpp
add_trust_analysis_command(
    ${CMAKE_CURRENT_SOURCE_DIR}/_cycles.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/_cycles
    trusted-cpp_unittest
    -Xclang -plugin-arg-trust -Xclang circleref-read=${CMAKE_CURRENT_SOURCE_DIR}/circleref.trust
)

# Для _example.cpp
add_trust_analysis_command(
    ${CMAKE_CURRENT_SOURCE_DIR}/_example.cpp
    ${CMAKE_CURRENT_SOURCE_DIR}/_example
    trusted-cpp_unittest
    -Xclang -plugin-arg-trust -Xclang circleref-disable
)

# Создадим цель для запуска тестов
add_custom_target(run_tests
    COMMAND ${CMAKE_CURRENT_SOURCE_DIR}/trusted-cpp_unittest
    DEPENDS trusted-cpp_unittest
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}
    COMMENT "Running trust unit tests"
)
