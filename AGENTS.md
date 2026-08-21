# AGENTS.md – Guidance for AI Agents

This file is intended for AI coding agents (LLM-based assistants, code completion tools, automated refactors, etc.) that interact with the **কোড.বাংলা (code.bangla)** programming language project. It is symlinked to `.github/copilot-instructions.md` and `CLAUDE.md` so all agents share one source of truth.

The goal is to help you:

- Understand the project structure and goals.
- Generate correct, consistent code and suggestions.
- Work autonomously and rigorously, regardless of which tool or model you are.
- Avoid common pitfalls when working with a Bangla-first language implemented in Rust.

This file makes no assumptions about which agent, model, or editor is reading it. Don't hardcode behavior to a specific tool's tool-calling conventions — treat "search the web", "fetch a URL", "read a file", "run a shell command" as generic capabilities every capable agent has some equivalent of, under whatever name your host exposes them.

---

## Project Overview

- **Name**: কোড.বাংলা (code.bangla) programming language (`code-bangla`)
- **Implementation language**: Rust
- **Surface language**: Bangla (keywords, commands, standard library APIs, diagnostics)
- **Execution modes**:
  - Interpreted: source → AST → IR → interpreter (no linker). `.বাংলা` files are executed directly by the interpreter, line by line.
  - Compiled: source → AST → IR → native code (Cranelift) → linked executable.
- **Design goals**:
  - The language is TypeScript-like, with all TypeScript features.
  - Ownership and borrowing concepts from Rust should be implemented.
  - Memory-safe, high-performance, native binaries.
  - No C source code; only Rust.
  - No VM/runtime dependency for compiled binaries (beyond system libs).
  - Fully Bangla user-facing interface (CLI, errors, docs).
  - Security over features.

Always assume:

- The **user-facing language** is Bangla.
- The **implementation** is Rust.
- Agents should reason in English internally, but produce Bangla where user-facing text is required.
- Agents are copilots, not the core developer — propose and implement changes, but treat the human as the decision-maker on design questions.

---

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
- `lang-codegen/` – Native codegen (Cranelift).
- `lang-runtime/` – Runtime support (panic handling, FFI helpers).
- `lang-stdlib/` – Standard library (in Bangla).
- `lang-lsp/` – Language server (LSP) implementation.
- `lang-cli/` – CLI binary (`বাংলা`).
- `tests/` – Test programs and harness.
- `docs/` – Additional documentation, tutorials.

When generating code or suggestions, respect this structure.

---

## Language Surface (Bangla)

For all user-facing content, use Bangla:

- Keywords: `ফাংশন`, `রিটার্ন`, `যদি`, `অন্যথা`, `ভেরিয়েবল`, etc.
- CLI commands: `চালাও`, `তৈরি করো`, `পরীক্ষা`, `সাজাও`, `সংস্করণ`, `সাহায্য`.
- Standard library functions: `দেখাও`, `পড়ো`, `দৈর্ঘ্য`, etc.
- Error messages and diagnostics: in Bangla.

Do **not** introduce English keywords or commands unless explicitly asked. The language design is Bangla-only on the surface. Follow Bangla Academy rules for spelling.

When you need to refer to language constructs internally (in reasoning or comments), you can use English equivalents, but generated code that users will see must be in Bangla.

Examples:

- Function: `ফাংশন মূল () { ... }`
- CLI: `বাংলা চালাও মূল.বাংলা`
- Error: `ত্রুটি: চলক 'x' সংজ্ঞায়িত নয়।`

Example Code:

```কোড.বাংলা
চলক বয়স = ১৬

ধ্রুবক নাম = "সমুদ্রী"

দেখাও(`আমার নাম ${নাম}, আমার বয়স ${বয়স}`)

যদি (বয়স >= ১৮) {
  প্রাপ্তবয়স্ক = হ্যা;
} নতুবা {
  প্রাপ্তবয়স্ক = না;
}
```

---

## Implementation Details (Rust)

All implementation code is Rust:

- Use modern Rust (edition 2024 or latest stable).
- Follow standard Rust style and conventions (`cargo fmt`, `cargo clippy`).
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
  - Lowers IR to Cranelift IR.
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

---

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

---

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

---

## Interpreter vs Compiler

The same frontend (lexer, parser, type checker) and IR are used for both modes.

- Interpreted mode:
  - Flow: source → AST → IR → interpreter.
  - No object files, no linker.
  - Slower, but fast iteration.
- Compiled mode:
  - Flow: source → AST → IR → codegen (Cranelift) → object files → linker → native executable.

---

## Agent Working Practices

These practices apply regardless of which agent or tool is executing them. Adapt them to whatever concrete tool-calling interface your host provides (shell access, file read/write, web search, etc.) — nothing here should be read as naming a specific product.

### General approach

- Keep going until the task is genuinely resolved; don't hand control back with loose ends unless you need a decision only the human can make.
- Prefer web search for researching over reasoning hallucination.
- Plan before acting. Break non-trivial work into a step-by-step todo list (see format below) and keep it updated as you go.
- Investigate the actual codebase before proposing a fix — read the relevant modules (`mod.rs`, `lib.rs`, crate roots), find the related `fn`/`struct`/`enum`/`trait` items, and confirm your understanding of the root cause before editing.
- Make small, incremental, testable changes rather than large rewrites.
- After changes, verify: run `cargo build`, `cargo test`, `cargo clippy`, `cargo fmt --check` as applicable, and re-read the diff.
- If a patch doesn't apply cleanly, re-read the current file state and reapply rather than guessing.
- State your next action in one concise sentence before taking it, then actually take it — don't describe an action you don't follow through on.
- When resuming a previous session ("continue", "resume"), check prior context for the last incomplete todo item and continue from there, explaining briefly what you're resuming.

### Research

- When you need current information about a crate, API, or language feature — because your training data may be stale or the codebase uses a specific version — use whatever web search/fetch capability is available to you rather than relying on memory.
- Prefer primary sources: [docs.rs](https://docs.rs), [The Rust Book](https://doc.rust-lang.org/book/), [users.rust-lang.org](https://users.rust-lang.org), the crate's own repository, and RFC/spec documents. Community sites ([Rust Reddit](https://reddit.com/r/rust), Stack Overflow) are useful for corroboration, not as primary authority.
- Follow relevant links from a fetched page when they clarify the API surface further, rather than stopping at the first result.
- Cross-check anything version-sensitive (API signatures, feature flags, MSRV) against the actual `Cargo.lock`/`Cargo.toml` in this repo.

### Todo list format

Use plain markdown, wrapped in triple backticks so it renders correctly:

```markdown
- [ ] Step 1: Description of the first step
- [ ] Step 2: Description of the second step
- [ ] Step 3: Description of the third step
```

Status markers:

- `[ ]` = Not started
- `[x]` = Completed
- `[-]` = Removed or no longer relevant

Update and redisplay the list as you complete steps; don't stop mid-list to ask what to do next unless you're genuinely blocked on a decision.

### Rust anti-patterns to avoid

Check your plan against these before implementing:

- Using `.clone()` instead of borrowing where borrowing would do.
- Overusing `.unwrap()`/`.expect()` in non-test code paths — prefer proper `Result`/`Option` handling (`thiserror`/`anyhow` where appropriate).
- Calling `.collect()` too early and losing iterator laziness.
- Writing `unsafe` without a clearly documented need and invariant.
- Over-abstracting with traits/generics where a concrete type would be clearer.
- Relying on global mutable state, which breaks testability.
- Heavy macro use that hides logic and makes code hard to debug or `cargo-expand`.
- Ignoring lifetime annotations in a way that produces confusing borrow errors downstream.
- Optimizing before correctness is established and tested.

If this project later grows a GUI (LSP client, playground, etc.), apply the same care to thread-safety there: keep UI updates on the main thread, use message-passing or the toolkit's async-safe primitives to reach it from worker threads, and prefer `Arc<Mutex<_>>`/`Arc<RwLock<_>>` with clear ownership over ad hoc shared state — but don't pull in toolkit-specific guidance until a toolkit is actually chosen.

### Debugging

- Prefer finding the root cause over patching symptoms.
- Use `RUST_BACKTRACE=1` for stack traces, `dbg!()` for quick inspection, and structured logging (`tracing` or `log`) for anything that needs to persist across a run.
- Use `cargo-expand` to debug macro-heavy code.
- Read tool/test output fully before deciding on a next step.

### Communication style

Be concise, direct, and specific about what you're doing and why. Say what tool or command you're about to run in one sentence, then run it.

Examples:

- "Checking docs.rs for the current `tokio::select!` signature before using it here."
- "Tests pass; adding an edge case for empty Bangla identifiers."
- "That `unwrap()` will panic on malformed input — switching to a `Result` with a proper diagnostic."

---

## User-Supplied Research URLs

Agents should fetch and read these (and follow directly relevant links within them) rather than treating the list as decorative.

- [Blang](https://github.com/sergev/blang) - This implements a similar compiler for The B language using Go.
- [TypeScript Go](https://github.com/microsoft/typescript-go)  - This implements a similar interpreter for The TypeScript language using Go.
- [bnlang](https://bnlang.dev/) - This is a similar project implemented in C++
- [Rust Documentaion](https://doc.rust-lang.org/stable/)
- [The Rust Reference](https://doc.rust-lang.org/stable/reference/index.html)
- [Cranelift Docs](https://github.com/bytecodealliance/wasmtime/tree/main/cranelift/docs)
- [ScriptC](https://github.com/vercel-labs/scriptc) - TypeScript to native compiler using LLVM IR
