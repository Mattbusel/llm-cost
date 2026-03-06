cmake_minimum_required(VERSION 3.16)
project(llm_cost_examples CXX)

set(CMAKE_CXX_STANDARD 17)
set(CMAKE_CXX_STANDARD_REQUIRED ON)

# No external dependencies — pure C++17

add_executable(count_tokens   examples/count_tokens.cpp)
target_include_directories(count_tokens   PRIVATE include)

add_executable(compare_models examples/compare_models.cpp)
target_include_directories(compare_models PRIVATE include)

add_executable(budget_guard   examples/budget_guard.cpp)
target_include_directories(budget_guard   PRIVATE include)
