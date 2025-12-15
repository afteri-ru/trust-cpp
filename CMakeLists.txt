cmake_minimum_required(VERSION 3.20)

project(trust-cpp VERSION 1.0.0 LANGUAGES C CXX)

# Используем clang++ напрямую, как в исходных Makefile
set(CMAKE_CXX_COMPILER clang++-21)

# Установим стандарт C++20, как в оригинальном Makefile
set(CMAKE_CXX_STANDARD 20)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# Добавим подпроекты
add_subdirectory(plugin)
add_subdirectory(test)
