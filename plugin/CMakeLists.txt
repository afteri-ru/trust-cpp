cmake_minimum_required(VERSION 3.20)

# Найдем необходимые компоненты
find_package(PkgConfig REQUIRED)
pkg_check_modules(YAML_CPP REQUIRED yaml-cpp)

# Установим пути к заголовочным файлам LLVM
set(LLVM_INCLUDE_DIRS "/usr/lib/llvm-21/include")
set(LLVM_LIBRARIES "-L/usr/lib/llvm-21/lib -lLLVM-21")
set(LLVM_DEFINITIONS "-D_GNU_SOURCE -DEXPERIMENTAL_KEY_INSTRUCTIONS -D__STDC_CONSTANT_MACROS -D__STDC_FORMAT_MACROS -D__STDC_LIMIT_MACROS")

# Добавим пути к заголовочным файлам
include_directories(${LLVM_INCLUDE_DIRS})
include_directories(${CMAKE_CURRENT_SOURCE_DIR}/..)  # Для memsafe.h в корне проекта

# Добавим определения компилятора
add_definitions(${LLVM_DEFINITIONS})

# Создадим цель для плагина clang
add_custom_command(
    OUTPUT memsafe_clang.so
    COMMAND clang++-21 -fPIC -shared -o memsafe_clang.so plugin/memsafe_clang.cpp `llvm-config-21 --cppflags --ldflags --system-libs --libs all` -std=c++20 -L/usr/lib/x86_64-linux-gnu -lyaml-cpp -I${CMAKE_CURRENT_SOURCE_DIR}/..
    DEPENDS memsafe_clang.cpp
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/..
    COMMENT "Building memsafe_clang.so plugin"
)

add_custom_target(memsafe_clang_target ALL DEPENDS memsafe_clang.so)

# Создадим псевдо-цель, чтобы CMake думал, что библиотека существует
add_library(memsafe_clang SHARED IMPORTED)
set_target_properties(memsafe_clang PROPERTIES
    IMPORTED_LOCATION ${CMAKE_CURRENT_SOURCE_DIR}/../memsafe_clang.so
)
add_dependencies(memsafe_clang memsafe_clang_target)

# Создадим псudo-цель для генерации файла circleref.memsafe
add_custom_target(circleref_memsafe ALL
    COMMAND clang++-21 -std=c++20 -ferror-limit=500 -I${CMAKE_CURRENT_SOURCE_DIR}/..
            -Xclang -load -Xclang $<TARGET_FILE:memsafe_clang>
            -Xclang -add-plugin -Xclang memsafe
            -Xclang -plugin-arg-memsafe -Xclang level=warning
            -Xclang -plugin-arg-memsafe -Xclang circleref-write=${CMAKE_CURRENT_SOURCE_DIR}/../test/circleref.memsafe
            -fsyntax-only test/_example.cpp
    DEPENDS memsafe_clang
    WORKING_DIRECTORY ${CMAKE_CURRENT_SOURCE_DIR}/..
    COMMENT "Generating circleref.memsafe"
)
