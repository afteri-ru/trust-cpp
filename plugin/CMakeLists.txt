# Найдем необходимые компоненты
find_package(PkgConfig REQUIRED)
pkg_check_modules(YAML_CPP REQUIRED yaml-cpp)

# Попробуем найти LLVM через find_package с компонентами, необходимыми для clang plugin
find_package(LLVM REQUIRED CONFIG)

# Создадим библиотеку плагина clang
add_library(memsafe_clang SHARED
    memsafe_clang.cpp
)

# Добавим пути к заголовочным файлам LLVM
target_include_directories(memsafe_clang PRIVATE
    ${LLVM_INCLUDE_DIRS}
    ${CMAKE_CURRENT_SOURCE_DIR}/..
    ${YAML_CPP_INCLUDE_DIRS}
)

# Добавим определения компилятора
target_compile_definitions(memsafe_clang PRIVATE ${LLVM_DEFINITIONS})

# Установим свойства цели
set_target_properties(memsafe_clang PROPERTIES
    CXX_STANDARD 20
    CXX_STANDARD_REQUIRED ON
    PREFIX ""
    SUFFIX ".so"
    LIBRARY_OUTPUT_DIRECTORY "${CMAKE_CURRENT_SOURCE_DIR}/.."
)

# Линкуем с LLVM библиотеками, которые необходимы для clang plugin
target_link_libraries(memsafe_clang
    LLVMCore
    LLVMSupport
    LLVMAnalysis
    LLVMTransformUtils
    ${YAML_CPP_LIBRARIES}
)

# Создадим псевдо-цель для генерации файла circleref.memsafe
# Используем BYPRODUCTS для лучшей интеграции с системой сборки CMake
add_custom_command(
    OUTPUT ${CMAKE_CURRENT_SOURCE_DIR}/../test/circleref.memsafe
    COMMAND ${CMAKE_CXX_COMPILER} -std=c++20 -ferror-limit=500 -I${CMAKE_CURRENT_SOURCE_DIR}/..
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
