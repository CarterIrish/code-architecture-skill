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

## Pattern Choices

These are architectural decisions where multiple approaches are equally valid in C++. Present the options during the interview and let the user choose.

### Project Organization

Present for any project beyond a single-file script.

**A) Flat src/** — All source files in one `src/` directory. No subdirectories. Simple, low overhead.

```
src/
├── main.cpp
├── app.cpp / app.h
├── utils.cpp / utils.h
```

Best for: Small projects (<10 source files), CLI tools, projects where the organization is obvious from filenames alone.

**B) Library + application split** — Core logic lives in a static library target, the executable is a thin wrapper that links against it. Tests also link against the library.
Best for: Any project where you want testable core logic. This is the recommended default for C++ projects because it makes unit testing trivial — but some users find it over-structured for small tools.

**C) Modular (subdirectories per feature)** — Each feature or subsystem gets its own directory with its own sources and optionally its own `CMakeLists.txt`. Modules expose public headers and link against each other explicitly.
Best for: Medium-to-large projects, projects with clear subsystem boundaries, projects that may grow into multiple libraries.

### Error Handling

Present when the project has operations that can fail (file I/O, network, parsing, database).

**A) Exceptions** — Standard C++ error handling. Throw on failure, catch at boundaries. Clean happy path, but hidden control flow.

```cpp
std::string readFile(const std::string& path) {
    std::ifstream f(path);
    if (!f) throw std::runtime_error("Cannot open: " + path);
    // ...
}
```

Best for: Applications (not libraries), users from Java/C#/Python backgrounds, projects where errors are truly exceptional.

**B) Error codes / std::optional / std::expected (C++23)** — Return success/failure explicitly. No hidden control flow, caller must handle the result.

```cpp
std::optional<std::string> readFile(const std::string& path) {
    std::ifstream f(path);
    if (!f) return std::nullopt;
    // ...
}
```

Best for: Libraries, performance-sensitive code, users who prefer explicit error paths, C-adjacent codebases. `std::expected` (C++23) gives the best of both — explicit returns with error information — but requires a recent compiler.

**C) Mixed** — Exceptions for truly unexpected failures (out of memory, corrupted state), return values for expected failures (file not found, invalid input). This is common in real-world C++ but requires discipline to keep consistent.
Best for: Pragmatic projects that don't want a religious commitment to either approach.

### Dependency Management

Present when the project uses at least one third-party library.

**A) FetchContent** — CMake downloads and builds dependencies at configure time. No separate install step. Dependencies are compiled as part of your project.
Best for: Projects with 1-3 dependencies, header-only or small libraries, users new to C++ dependency management (simplest setup).

**B) vcpkg** — Microsoft's C++ package manager. Install packages globally or per-project with a manifest (`vcpkg.json`). Large ecosystem, handles transitive dependencies.
Best for: Projects with many dependencies, cross-platform projects, users who want `apt install`-style convenience for C++ libraries.

**C) Vendoring (third_party/)** — Copy dependency source directly into your repo. No external tools, no network needed at build time.
Best for: Projects that need hermetic builds, environments with restricted network access, users who want maximum control over dependency versions. Harder to update.

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
