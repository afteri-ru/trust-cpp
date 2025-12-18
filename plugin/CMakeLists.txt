# Создадим библиотеку плагина clang
add_library(trusted-cpp_clang SHARED
    trusted-cpp_clang.cpp
)

# Настроим плагин
target_include_directories(trusted-cpp_clang PRIVATE
    ${LLVM_INCLUDE_DIRS}
    ${CMAKE_CURRENT_SOURCE_DIR}/..
    ${YAML_CPP_INCLUDE_DIRS}
)

target_compile_definitions(trusted-cpp_clang PRIVATE ${LLVM_DEFINITIONS})

set_target_properties(trusted-cpp_clang PROPERTIES
    PREFIX ""
    SUFFIX ".so"
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/.."
)

# Используем общую функцию для установки свойств
set_common_target_properties(trusted-cpp_clang)

target_link_libraries(trusted-cpp_clang
    LLVMCore
    LLVMSupport
    LLVMAnalysis
    LLVMTransformUtils
    ${YAML_CPP_LIBRARIES}
)

# Создадим псевдо-цель для генерации файла circleref.trust
add_custom_command(
    OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/../test/circleref.trust
    COMMAND ${CMAKE_CXX_COMPILER} -std=c++20 -ferror-limit=500 -I${CMAKE_CURRENT_SOURCE_DIR}/../test
            -Xclang -load -Xclang $<TARGET_FILE:trusted-cpp_clang>
            -Xclang -add-plugin -Xclang trust
            -Xclang -plugin-arg-trust -Xclang level=warning
            -Xclang -plugin-arg-trust -Xclang circleref-write=${CMAKE_CURRENT_SOURCE_DIR}/../test/circleref.trust
            -fsyntax-only ${CMAKE_CURRENT_SOURCE_DIR}/../test/_example.cpp
    DEPENDS trusted-cpp_clang
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/..
    COMMENT "Generating circleref.trust"
)

add_custom_target(circleref_trust ALL
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/../test/circleref.trust
)
