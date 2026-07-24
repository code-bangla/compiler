# AGENTS.md – Guidance for AI Agents

This file is intended for AI coding agents (LLM-based assistants, code completion tools, automated refactors, etc.) that interact with the **Bangla programming language** project.

The goal is to help you:

- Understand the project structure and goals.
- Generate correct, consistent code and suggestions.
- Avoid common pitfalls when working with a Bangla-first language implemented in Rust.

## Project Overview

- **Name**: Bangla programming language (`bangla-lang`)
- **Implementation language**: Rust
- **Surface language**: Bangla (keywords, commands, standard library APIs, diagnostics)
- **Execution modes**:
  - Interpreted: source → AST → IR → interpreter (no linker).
  - Compiled: source → AST → IR → native code (Cranelift/LLVM) → linked executable.
- **Design goals**:
  - Memory-safe, high-performance, native binaries.
  - No C source code; only Rust.
  - No VM/runtime dependency for compiled binaries (beyond system libs).
  - Fully Bangla user-facing interface (CLI, errors, docs).

Always assume:

- The **user-facing language** is Bangla.
- The **implementation** is Rust.
- AI agents should reason in English internally, but produce Bangla where user-facing text is required.

## Repository Structure

Expected top-level layout:

- `README.md` – User-facing overview (in Bangla).
- `CONTRIBUTING.md` – Contribution guidelines (in Bangla).
- `VISION.md` – Long-term vision (in Bangla).
- `spec.md` – Language specification (in Bangla).
- `AGENTS.md` – This file (English).
- `Cargo.toml` – Rust workspace manifest.
- `lang-core/` – Lexer, parser, type checker, AST.
- `lang-ir/` – Intermediate representation used by both interpreter and compiler.
- `lang-interpret/` – Interpreter for IR.
- `lang-codegen/` – Native codegen (Cranelift/LLVM).
- `lang-runtime/` – Runtime support (panic handling, FFI helpers).
- `lang-stdlib/` – Standard library (in Bangla).
- `lang-lsp/` – Language server (LSP) implementation.
- `lang-cli/` – CLI binary (`বাংলা`).
- `tests/` – Test programs and harness.
- `docs/` – Additional documentation, tutorials.

When generating code or suggestions, respect this structure.

## Language Surface (Bangla)

For all user-facing content, use Bangla:

- Keywords: `ফাংশন`, `রিটার্ন`, `যদি`, `অন্যথা`, `ভেরিয়েবল`, etc.
- CLI commands: `চালাও`, `তৈরি করো`, `পরীক্ষা করো`, `সাজাও`, `সংস্করণ`, `সাহায্য`.
- Standard library functions: `লেখো`, `পড়ো`, `দৈর্ঘ্য`, etc.
- Error messages and diagnostics: in Bangla.

Do **not** introduce English keywords or commands unless explicitly asked. The language design is Bangla-only on the surface.

When you need to refer to language constructs internally (in reasoning or comments), you can use English equivalents, but generated code that users will see must be in Bangla.

Examples:

- Function: `ফাংশন মূল () { ... }`
- CLI: `বাংলা চালাও মূল.বাংলা`
- Error: `ত্রুটি: চলক 'x' সংজ্ঞায়িত নয়।`

Example Code:

```কোড.বাংলা
চলক বয়স = ১৬

ধ্রুবক নাম = "সমুদ্রী"

দেখাও(`আমার নাম ${নাম}, আমার বয়স ${বয়স}`)

যদি (বয়স >= ১৮) {
  প্রাপ্তবয়স্ক = হ্যা;
} নতুবা {
  প্রাপ্তবয়স্ক = না;
}

```

## Implementation Details (Rust)

All implementation code is Rust:

- Use modern Rust (edition 2024 or latest stable).
- Follow standard Rust style and conventions.
- Keep internal names (functions, types, modules) in English for clarity.
- Use Bangla only where interfacing with the Bangla language surface:
  - Keyword tables.
  - CLI strings.
  - Error message templates.
  - Standard library API names exposed to Bangla code.

Key crates:

- `lang-core`:
  - Lexer: UTF-8 input, Bangla keywords, Unicode-aware.
  - Parser: builds AST.
  - Type checker: name resolution, type checking.
- `lang-ir`:
  - Defines the IR instructions and structures.
  - Used by both interpreter and compiler.
- `lang-interpret`:
  - Implements a simple stack/heap-based VM over IR.
- `lang-codegen`:
  - Lowers IR to Cranelift/LLVM IR.
  - Emits object files and orchestrates linking.
- `lang-runtime`:
  - Minimal runtime helpers (panic, FFI, entry points).
- `lang-stdlib`:
  - Standard library modules exposed to Bangla code.
- `lang-lsp`:
  - Language server using LSP (diagnostics, go-to-def, hover, etc.).
- `lang-cli`:
  - `বাংলা` binary using `clap` with Bangla commands and help text.

When modifying or generating Rust code:

- Keep internal logic in English.
- Keep user-facing strings (CLI, errors, keywords) in Bangla.

## Unicode and Bangla Text Handling

Important rules for working with Bangla source text:

- Source files are UTF-8, preferably NFC-normalized.
- Identifiers can contain Bangla letters, digits, and `_`, but must not start with a digit.
- Apply Unicode security best practices (similar to UTS #39):
  - Avoid confusable characters in identifiers.
  - Be cautious with mixed-script identifiers.
- Use a Unicode-aware segmentation library for:
  - Grapheme clusters (for editor operations).
  - Line breaking and wrapping (for formatters, diagnostics).

When generating diagnostics:

- Use byte offsets or line/column positions.
- Ensure error snippets preserve Bangla text correctly.
- Do not split graphemes in the middle when truncating or highlighting code.

## CLI Behavior

The CLI binary is named `বাংলা` and supports Bangla subcommands:

- `চালাও` – Run in interpreted mode.
- `তৈরি করো` – Compile to native executable.
- `পরীক্ষা করো` – Type-check only.
- `সাজাও` – Format code.
- `সংস্করণ` – Show version.
- `সাহায্য` – Show help.

Implementation notes:

- Implemented in Rust using `clap`.
- All command names, help texts, and errors are in Bangla.
- No English aliases are exposed in the final design (Bangla-only interface).

When generating CLI examples or docs for users:

- Always use Bangla commands.
- Do not suggest English equivalents unless explicitly requested.

## Interpreter vs Compiler

The same frontend (lexer, parser, type checker) and IR are used for both modes.

- Interpreted mode:
  - Flow: source → AST → IR → interpreter.
  - No object files, no linker.
  - Slower, but fast iteration.

- Compiled mode:
  - Flow: source → AST → IR → codegen (Cranelift/LLVM) → object files → linker
