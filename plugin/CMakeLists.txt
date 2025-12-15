# Создадим библиотеку плагина clang
add_library(memsafe_clang SHARED
    memsafe_clang.cpp
)

# Настроим плагин
target_include_directories(memsafe_clang PRIVATE
    ${LLVM_INCLUDE_DIRS}
    ${CMAKE_CURRENT_SOURCE_DIR}/..
    ${YAML_CPP_INCLUDE_DIRS}
)

target_compile_definitions(memsafe_clang PRIVATE ${LLVM_DEFINITIONS})

set_target_properties(memsafe_clang PROPERTIES
    PREFIX ""
    SUFFIX ".so"
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/.."
)

# Используем общую функцию для установки свойств
set_common_target_properties(memsafe_clang)

target_link_libraries(memsafe_clang
    LLVMCore
    LLVMSupport
    LLVMAnalysis
    LLVMTransformUtils
    ${YAML_CPP_LIBRARIES}
)

# Создадим псевдо-цель для генерации файла circleref.memsafe
add_custom_command(
    OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/../test/circleref.memsafe
    COMMAND ${CMAKE_CXX_COMPILER} -std=c++20 -ferror-limit=500 -I${CMAKE_CURRENT_SOURCE_DIR}/../test
            -Xclang -load -Xclang $<TARGET_FILE:memsafe_clang>
            -Xclang -add-plugin -Xclang memsafe
            -Xclang -plugin-arg-memsafe -Xclang level=warning
            -Xclang -plugin-arg-memsafe -Xclang circleref-write=${CMAKE_CURRENT_SOURCE_DIR}/../test/circleref.memsafe
            -fsyntax-only ${CMAKE_CURRENT_SOURCE_DIR}/../test/_example.cpp
    DEPENDS memsafe_clang
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/..
    COMMENT "Generating circleref.memsafe"
)

add_custom_target(circleref_memsafe ALL
    DEPENDS ${CMAKE_CURRENT_SOURCE_DIR}/../test/circleref.memsafe
)
