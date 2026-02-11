# GitHub Copilot instructions for this repository

Purpose
- Assist contributors by generating code, tests, docs, and CI changes that match the repository's existing style and build system.
- Avoid speculative design: prefer minimal, buildable, and reviewable changes.

Repository context
- Primary languages: C++, CMake, assorted build and shader files.
- Build system: CMake (edit `CMakeLists.txt` when adding/removing sources).
- Keep cross-platform behavior and existing public APIs intact unless asked to modernize.

General rules for suggestions
- Always produce complete, compilable snippets or complete file contents; include necessary includes, namespaces, and build changes (CMake) when adding files.
- Match existing code style and patterns. Use the surrounding code as the authority for formatting, naming, error handling, and memory-management patterns.
- Make minimal, incremental changes. If a change touches many files or is architectural, propose a short plan and break the work into PR-sized steps.
- Never add secrets, credentials, API keys, or proprietary data to the repository.

Build & test
- Verify suggestions build locally with the project's build steps (edit `CMakeLists.txt` appropriately). Recommend running __Build > Build Solution__ (or equivalent CMake build) before merging.
- If the change involves behavior, include or update tests and run them via __Test Explorer__ or the repository's test command.

Commit & PR guidance
- Use imperative, concise commit messages (e.g., "Fix crash when loading level", "Add unit tests for X").
- When changing public behavior or API, add an entry to `CHANGELOG.md` or include a short changelog note in the PR description.
- Provide a clear PR description: motivation, summary of changes, how to test, and risks.

Security & licensing
- Respect upstream licenses and existing notices in source files. Do not remove or alter license headers unless instructed.
- Flag potential security issues (unsafe file I/O, unchecked user input, insecure dependencies) and propose secure alternatives.

When unsure
- Ask a focused clarifying question before generating code (target platform, required C++ standard, expected behavior, required compatibility).
- If a requested change cannot be made safely without more context, provide a short plan with options, trade-offs, and required follow-ups.

Formatting output for reviewers
- Provide full file content for new or replaced files.
- For patches, show the changed file with context and the exact edits required.
- When recommending CMake edits, include the full modified `CMakeLists.txt` snippet to apply.

If you are a human contributor reading these instructions: follow them when interacting with Copilot and when reviewing Copilot-suggested changes.