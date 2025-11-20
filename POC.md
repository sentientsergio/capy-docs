# Capy (Boost.RunTimeServices) Documentation POC

Purpose: demonstrate a concrete, high-signal doc plan and sample content that improves on the current RTS docs and can guide future library docs.

## Goals
- Provide a Divio-style structure (Overview → Quickstart → How-to → Reference → Explanations).
- Show minimal working code with build/link steps.
- Call out C++ realities: lifetimes, thread safety, error model, template constraints, integration.
- Keep tone concise and specific; avoid filler.

## Current outline score (RTS docs)
- Structure 2/10; Quickstart 0/10; How-to 0/10; Examples 0/10; Ownership 0/10; Error handling 0/10; Thread safety 0/10; Templates 0/10; Integration 1/10 (compiler list only); Tone 6/10; Cross-links 1/10. Overall ~1–2/10.

## Proposed outline (Capy future state)
- Overview: what/why, target platforms (C++11; exceptions/RTTI optional), positioning among Boost libs, tested compilers/standards.
- Quickstart: minimal service registration + lifecycle; build/link commands.
- Concepts: service model, two-phase init (`emplace` → optional `start`/`stop`), type-based lookup and `key_type`, lifetime (destroyed reverse creation), error model (duplicates throw `std::invalid_argument`, missing type throws `std::bad_typeid`), thread safety (const ops thread-safe; mutating ops not concurrent; `stop` is idempotent and may be concurrent).
- How-to guides:
  - Register/access services: `emplace`, `insert`, `try_emplace`, `use`, `find`/`get`, `key_type`.
  - Inject dependencies: `rts::invoke` to auto-wire stored parts.
  - Manage lifecycle: `start`/`stop`/`join` ordering; handling parts without `start`/`stop`.
  - Exception-free builds (if needed): prefer error_code-returning APIs; note throw sites.
  - Embedded constraints: no RTTI/exceptions; footprint and build flags.
  - Compression helpers (brotli/zlib) if shipped: setup, usage, error surfaces.
- Reference: mrdocs-generated API; cross-linked from guides.
- Troubleshooting/FAQ: duplicate insertion, missing types, calling `start` twice, RTTI-free builds.
- Integration appendix: CMake usage, compiler standard, optional package managers, CI matrix link.

## Sample Quickstart (draft)
```cpp
#include <boost/rts/application.hpp>

struct logger {
    void start() {} // optional hook
    void stop() {}  // optional hook
};

int main() {
    boost::rts::application app;
    auto& log = app.emplace<logger>();
    app.start();    // calls logger::start if present
    // ... use log ...
    app.stop();     // idempotent; reverse order
    app.join();     // wait for stop completion
}
```

Build/link:
```cmake
find_package(Boost 1.86 COMPONENTS rts REQUIRED)
add_executable(capy_hello main.cpp)
target_link_libraries(capy_hello PRIVATE Boost::rts)
set_target_properties(capy_hello PROPERTIES CXX_STANDARD 11)
```

Notes:
- Stored objects live until the container is destroyed; destruction is reverse creation.
- Only one `start` call is allowed; `stop` is safe to call concurrently and multiple times.
- `get<T>` throws `std::bad_typeid` if missing; duplicate insertion throws `std::invalid_argument`.
- `const` member calls are thread-safe; mutating ops (`emplace`/`insert`/`try_emplace`) must not run concurrently on the same container.

## Anticipated score after implementation
- Structure 9/10; Quickstart 8/10; How-to 8/10; Examples 8/10; Ownership 8/10; Error handling 7/10; Thread safety 7/10; Templates/constraints 7/10; Integration 8/10; Tone 8/10; Cross-links 8/10. Overall ~8/10 once populated and tested.

## Next actions
- Fill out pages in `doc/modules/ROOT/pages`: index (Overview), quickstart, guides, concepts, FAQ; link them in nav.
- Fix doc build (Antora + mrdocs) and wire reference into nav.
- Add tested snippets/examples and CI checks to keep them compiling.
