cmake_minimum_required(VERSION 3.20)

project(trust-cpp VERSION 1.0.0 LANGUAGES C CXX)

# Используем clang++ напрямую, как в исходных Makefile
set(CMAKE_CXX_COMPILER clang++-21)

# Установим стандарт C++20, как в оригинальном Makefile
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Включим параллельную сборку
set(CMAKE_BUILD_PARALLEL_LEVEL 4)

# Добавим флаги оптимизации
set(CMAKE_CXX_FLAGS "${CMAKE_CXX_FLAGS} -O3")

# Добавим подпроекты
add_subdirectory(plugin)
add_subdirectory(test)
