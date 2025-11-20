# Prompt Tool: High-Signal C++ Library Documentation (Capy/RTS and beyond)

Use this template to steer AI-written docs so they read like careful human work, not generic prose. Apply it when drafting new pages, reviewing AI output, or asking for revisions.

## How to use
1) Set the context: describe the library, primary audiences, C++ standard/constraints (e.g., exceptions off/RTTI off), and top 3 use-cases.  
2) Paste the relevant API or code excerpt (or link to header).  
3) Ask the model to produce or revise documentation following the guardrails and checklist below.  
4) Self-check with the checklist; iterate until each item is satisfied.

## Guardrails (must follow)
- Be specific: every claim must tie to an API, behavior, or code fact. No filler, no empty adjectives.  
- Show, don’t gush: include small, compilable snippets for each major feature; prefer tested or testable code.  
- Address C++ realities: spell out ownership/lifetimes, thread-safety, error handling (exceptions vs error_code), and template constraints/concepts.  
- Integration clarity: include install/build/link steps (CMake, required standards/flags, dependencies).  
- Tone: concise, professional, and instructive; avoid signature-parroting (“takes int, returns bool”) without insight.  
- Navigation: place content into Overview → Quickstart → How-to guides → Reference → Explanations; cross-link appropriately.

## RTS/Capy current docs score (outline)
- Structure 2/10 (landing + ref link only), Quickstart 0/10, Task guides 0/10, Examples 0/10, Ownership 0/10, Error handling 0/10, Thread safety 0/10, Templates 0/10, Integration 1/10 (compiler list only), Tone 6/10 (concise but sparse), Cross-links 1/10. Overall ~1–2/10.

## Capy future-state outline (P0 content)
- Overview: what/why, target environments (embedded-friendly, exceptions/RTTI optional), positioning vs other Boost libs; tested compilers/standards.
- Quickstart: minimal service registration and lifecycle.
  - Code:
    ```cpp
    #include <boost/rts/application.hpp>

    struct logger {
        void start() {} // optional
        void stop() {}  // optional
    };

    int main() {
        boost::rts::application app;
        auto& log = app.emplace<logger>();
        app.start();           // calls logger::start if present
        // ... work with log ...
        app.stop();            // idempotent; calls stop in reverse order
        app.join();            // wait until stop completes
    }
    ```
  - Build/link:
    ```cmake
    find_package(Boost 1.86 COMPONENTS rts REQUIRED)
    add_executable(capy_hello main.cpp)
    target_link_libraries(capy_hello PRIVATE Boost::rts)
    set_target_properties(capy_hello PROPERTIES CXX_STANDARD 11)
    ```
  - Notes: no exceptions/RTTI required; `start` callable once; `stop` thread-safe/idempotent.
- Concepts: service = stored object; two-phase init (`emplace` -> optional `start`/`stop`); type-based lookup (`get`/`find`); `key_type` to alias storage; lifetime = stored until container destruction, destroyed reverse-creation; error model (duplicates throw `std::invalid_argument`, missing type throws `std::bad_typeid`); thread-safety (const ops thread-safe, mutating ops and `emplace` not concurrent; `stop` may be concurrent).
- How-to guides (task-focused):
  - Register/access services with `emplace`, `insert`, `try_emplace`, `use`, and `key_type`.
  - Inject dependencies and invoke callables (`rts::invoke`) to auto-wire stored types.
  - Manage lifecycle: start/stop/join ordering; handling parts without `start`/`stop`.
  - Exception-free builds: rely on error_code-returning APIs if present; note duplicate/missing throw sites.
  - Embedded constraints: working without RTTI/exceptions; footprint/compilation tips.
  - Compression helpers (brotli/zlib) if kept: setup, examples, error surfaces.
- Reference: mrdocs-generated API; cross-link from guides/concepts.
- Troubleshooting/FAQ: duplicate insertion, missing type lookups, start called twice, mixing RTTI-free builds.
- Integration appendix: supported compilers/standards, optional package managers (Conan/vcpkg), CI matrix link.

## Anticipated score for future outline
- Structure 9/10; Quickstart 8/10 (needs tested snippet); Task guides 8/10; Examples 8/10; Ownership 8/10; Error handling 7/10; Thread safety 7/10; Templates/constraints 7/10; Integration 8/10; Tone 8/10; Cross-links 8/10. Overall ~8/10 once populated and tested.

## Task prompt template
```
You are writing documentation for <LIBRARY> (C++). Audience: <who>. Constraints: <C++ standard>, exceptions <on/off>, RTTI <on/off>, platforms <...>.
Produce/updating: <page or section>. Scope: <features/APIs>. Source: <header snippet or link>.

Apply the rules:
- Structure: Overview (what/why) → Quickstart (minimal working example + build/link steps) → How-to(s) for the main tasks → Concepts (mental model, lifetimes, error model, thread safety) → Reference pointers.
- Examples: Provide copy-pasteable C++ snippets for each major task. Keep them minimal but complete (includes + main() or function scope as needed).
- Ownership/lifetimes: State what must outlive what; warn about pitfalls.
- Thread safety: State what can be called concurrently and what requires external sync. Mention callback/executor context if relevant.
- Error model: List how failures are reported (exceptions, error_code, status returns) and typical codes/exceptions.
- Template requirements: Describe constraints/Concepts on type parameters in plain language.
- Integration: Show CMake usage (target_link_libraries), required definitions/flags, and supported compilers/standards.
- Anti-slop: No generic praise, no restating signatures without insight. Every paragraph must add value.

Return: Markdown/adoc consistent with Boost docs; concise, concrete, cross-linked where helpful.
```

## Checklist for acceptance (deny until all pass)
- Structure: Has Overview, Quickstart, task-centric How-to(s), Concepts/Notes, and Reference links. Navigation is clear.  
- Quickstart: Minimal working program with build/link steps; compiles under stated constraints.  
- Task guides: Stepwise instructions for real tasks, not toy code; includes boilerplate and troubleshooting hints.  
- Examples: Each major API/feature has a focused, runnable snippet; variables are initialized; no pseudocode.  
- Ownership/lifetimes: Explicit statements and warnings where relevant.  
- Error handling: Mechanism and expected errors/codes/exceptions stated for failing operations.  
- Thread safety/concurrency: What is safe concurrently, and what isn’t.  
- Templates/constraints: Requirements on type parameters are explained.  
- Integration: CMake/link flags, compiler standard, and dependencies are documented.  
- Tone/clarity: Plain, professional, tight prose; no filler or AI-like generalities.  
- Cross-links: Points to reference/API pages for details; avoids duplicating signatures as prose.
