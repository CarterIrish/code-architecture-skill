# C++ / CMake Architecture Reference

## Common Architectural Patterns

### Header / Source Separation
The foundational C++ convention:
- **Headers (.h / .hpp)** — Declarations, interfaces, class definitions.
- **Source (.cpp)** — Implementations.
- Use include guards (`#pragma once` or `#ifndef`) in every header.
- Keep headers minimal — forward-declare where possible to reduce compile times.

### Library + Application Split
Separate reusable logic from the executable entry point:
- Core logic lives in a library target (static or shared).
- The application (CLI, GUI) is a thin wrapper that links against the library.
- Makes unit testing trivial — tests link against the same library without needing to invoke the executable.

Best for: CLI tools, utilities, any project where you want testable core logic.

### Modular / Component Architecture
Organize by feature or subsystem, not by file type:
- Each module gets its own directory with its own headers, sources, and optionally its own CMakeLists.txt.
- Modules expose a public interface through a single header or a small set of headers.
- Inter-module dependencies are explicit via CMake's `target_link_libraries`.

Best for: Medium-to-large projects, projects that may grow over time.

### PIMPL (Pointer to Implementation)
Hide implementation details behind an opaque pointer:
- Reduces compile-time dependencies (changing implementation doesn't trigger recompilation of dependents).
- Keeps public headers clean.
- Trade-off: adds indirection and heap allocation.

Best for: Library APIs, projects where compile time is a concern.

## CMake Project Structure

### Small Project (single target)
```
project-root/
├── CMakeLists.txt          # Top-level: project(), add_executable()
├── src/
│   ├── main.cpp            # Entry point
│   ├── app.cpp             # Core application logic
│   └── app.h
├── include/                # Public headers (if building a library)
│   └── project/
├── tests/
│   ├── CMakeLists.txt
│   └── test_app.cpp
├── build/                  # Out-of-source build (gitignored)
└── README.md
```

### Medium Project (library + application + tests)
```
project-root/
├── CMakeLists.txt          # Top-level: project(), cmake_minimum_required(), add_subdirectory()
├── src/
│   ├── CMakeLists.txt      # Defines library and executable targets
│   ├── core/               # Core library module
│   │   ├── core.cpp
│   │   └── core.h
│   ├── utils/              # Utility module
│   │   ├── utils.cpp
│   │   └── utils.h
│   └── main.cpp            # Application entry point
├── include/                # Public headers for library consumers
│   └── project/
│       └── core.h
├── tests/
│   ├── CMakeLists.txt      # add_executable() for test targets, link against core lib
│   ├── test_core.cpp
│   └── test_utils.cpp
├── third_party/            # Vendored or FetchContent dependencies
├── cmake/                  # Custom CMake modules/find scripts
├── build/
└── README.md
```

## CMake Best Practices

### Modern CMake (Target-Based)
- Use `target_include_directories()`, `target_link_libraries()`, `target_compile_features()` instead of global `include_directories()` or `add_definitions()`.
- Specify `PUBLIC`, `PRIVATE`, `INTERFACE` to control dependency propagation.
- Set `CMAKE_CXX_STANDARD` or use `target_compile_features(mylib PUBLIC cxx_std_17)`.

### Dependency Management
- **FetchContent** — Pull dependencies at configure time. Good for header-only or small libraries.
- **find_package()** — For system-installed libraries or vcpkg/Conan-managed packages.
- **Vendoring (third_party/)** — Copy source into the repo. Simple but harder to update.
- **vcpkg / Conan** — Package managers for C++. Good for projects with many dependencies.

Recommendation: FetchContent for small dependency counts, vcpkg for larger projects.

### Testing
- **GoogleTest** — Industry standard, integrates well with CMake via FetchContent.
- **Catch2** — Header-only option, simpler setup, good for smaller projects.
- Use `enable_testing()` and `add_test()` so `ctest` works out of the box.

## Common Pitfalls (C++ / CMake)

- **Global CMake commands**: `include_directories()`, `link_libraries()`, `add_definitions()` pollute the global scope. Always use target-specific versions.
- **In-source builds**: Always build out-of-source (`cmake -B build`). Add `/build` to `.gitignore`.
- **Header dependency explosion**: One change triggers recompilation of everything. Use forward declarations, PIMPL, and minimal includes.
- **No separation of library and executable**: Putting everything in `main.cpp` makes it untestable. Extract logic into a library target.
- **Ignoring compiler warnings**: Use `-Wall -Wextra -Wpedantic` (GCC/Clang) or `/W4` (MSVC). Treat warnings as errors in CI with `-Werror`.
- **Raw pointers everywhere**: Use `std::unique_ptr` and `std::shared_ptr` for ownership semantics. Raw pointers are fine for non-owning references.
- **Not using `std::` algorithms and containers**: Prefer standard library containers and algorithms over hand-rolled data structures unless you have a measured performance reason.
