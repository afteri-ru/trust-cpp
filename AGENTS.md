## Mandatory Rules for this Project
- Development is done in C++ (using C is only allowed in cases where C++ is not applicable, such as when handling hardware interrupts)
- Using compiler clang++-20 or above
- Always add or re-introduce tests for new or changed code, even if no one has requested them.
- Always run unit tests after completing a task or before committing changes.
- Do not create missing files unless their creation is specified in the current task.

## CodeStyle Rules
- Always use spaces for indentation, 4 spaces wide.
- Use camelCase for variable names and the 'm_' prefix for class fields.
- Explain your reasoning before submitting code.
- Focus on code readability and maintainability.
- Upon task completion, the final report should include the model name, the initial task, and all summary statistics for interactions with the model.

## File Rules
- `memsafe_clang.cpp` is a clang plugin designed for compilation into a dynamic library.
- `memsafe_test.cpp` contains tests created using the Google Test Framework, designed to be compiled into an executable program.
