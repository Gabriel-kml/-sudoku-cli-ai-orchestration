# SUD-0: Development of a Robust Sudoku CLI Tool

## Epic Overview

This project involves building a high-performance, Node.js-based Command Line Interface (CLI) using TypeScript to solve, validate, and generate Sudoku puzzles. The tool will handle 81-character string inputs (using digits, `0`, or `.`) and files, providing a deterministic generation engine based on seeds and difficulty levels.

## Epic Success Criteria

- A user can run `sudoku solve --input <string/file>` and receive a valid solution or an error for unsolvable boards
- A user can run `sudoku validate --input <string/file>` to check board validity
- A user can run `sudoku generate --difficulty medium --seed 42` and get a reproducible board
- The CLI passes all smoke tests and unit tests with >90% coverage

## Epic Definition of Done (DoD)

- All user stories (SUD-1 through SUD-6) are completed and merged
- Codebase is fully type-safe (no `any` types)
- CLI is globally linkable via `npm link`
- Documentation (README) includes setup and usage instructions
- CI pipeline passes for all PRs

---

## BACKLOG_PACK

### Guardrails

- **Test-Driven Development**: Write unit tests for core logic before implementation
- **Small Diffs**: Keep commits and PRs focused on single story logic
- **Environment**: Target Node.js >= 20. Use ESM
- **Deterministic Logic**: The generator must produce the same puzzle for the same seed/difficulty
- **No UI**: Strictly CLI-based output (no browsers or heavy GUI frameworks)

## SUD-1: Project Scaffolding & CI Pipeline

Initialize the TypeScript project, configure the build system, and set up a GitHub Actions workflow for automated testing and type-checking.

### Acceptance Criteria

- Project initializes with `npm init` and `tsconfig.json`
- CLI framework (e.g., `commander` or `yargs`) is installed and basic "Hello World" command works
- CI workflow runs on every push to `main` or PR

### Definition of Done

- `npm run build` and `npm run test` execute without errors in the CI environment

### Implementation Notes

- Use `tsx` for development and `vitest` for testing
- Ensure `bin` field is set in `package.json`

### Files Likely Impacted

- `package.json`
- `tsconfig.json`
- `.github/workflows/ci.yml`
- `src/index.ts`

### Verification Commands

```bash
npm run lint
npm run build
node ./dist/index.js --help
```

## SUD-2: Input Parsing & Validation Core

Implement a robust parser that converts 81-character strings (digits, `0`, or `.`) or file paths into a 2D array or typed Board object. Create a validator to check Sudoku rules (rows, columns, 3x3 grids).

### Acceptance Criteria

- Supports input strings like `53..7....6..195.....`
- Correctly identifies invalid boards (duplicate numbers in a row/col/block)
- Handles file reading for the `--input` flag

### Definition of Done

- Unit tests cover valid, invalid, and malformed (wrong length) inputs

### Implementation Notes

- Use a flat `Array(81)` or `number[9][9]` internally
- Treat `.` and `0` as empty cells

### Files Likely Impacted

- `src/core/parser.ts`
- `src/core/validator.ts`
- `src/types.ts`

### Verification Commands

```bash
vitest src/core/parser.spec.ts
```

---

## SUD-3: The Solver Engine (Backtracking)

Implement the `sudoku solve` command using an efficient backtracking algorithm.

### Acceptance Criteria

- Accepts a board via the parser
- Returns a fully solved 81-char string if solvable
- Exits with a non-zero code and helpful message if unsolvable

### Definition of Done

- Passes "World's Hardest Sudoku" test cases within < 100ms

### Implementation Notes

- Optimize by finding the cell with the fewest possibilities first (MRV heuristic) if speed is an issue

### Files Likely Impacted

- `src/core/solver.ts`
- `src/commands/solve.ts`

### Verification Commands

```bash
sudoku solve --input "..."
```

---

## SUD-4: Deterministic Generator

Implement the `sudoku generate` command using a seed-based Random Number Generator (RNG).

### Acceptance Criteria

- Generates puzzles for easy, medium, and hard
- Uses a provided `--seed` to ensure the same puzzle is generated every time
- Puzzles must have a unique solution

### Definition of Done

- Generating with `--seed 42` twice results in identical strings

### Implementation Notes

- Use a library like `seedrandom`
- Start with a full board, shuffle using the seed, and remove numbers while ensuring the solver still finds a unique solution

### Files Likely Impacted

- `src/core/generator.ts`
- `src/commands/generate.ts`

### Verification Commands

```bash
sudoku generate --difficulty easy --seed 123
```

---

## SUD-5: CLI Integration & Smoke Tests

Connect the core engines to the CLI framework and implement end-to-end (E2E) smoke tests.

### Acceptance Criteria

- `sudoku validate` output is user-friendly (e.g., "Board is valid")
- `sudoku solve` outputs the board string to stdout
- Smoke tests run the actual compiled binary against sample inputs

### Definition of Done

- Shell scripts or Vitest E2E tests pass for all three main commands

### Implementation Notes

- Use `execa` or `child_process` to test the CLI output in Vitest

### Files Likely Impacted

- `src/cli.ts`
- `tests/smoke.spec.ts`

### Verification Commands

```bash
npm run test:smoke
```

---

## SUD-6: Documentation & AI Readiness

Finalize the README and add metadata files to help AI assistants understand the project structure (e.g., `.cursorrules` or `ARCHITECTURE.md`).

### Acceptance Criteria

- README contains clear installation and usage examples
- Project includes a `CONTRIBUTING.md` or AI-context file describing the board format

### Definition of Done

- All documentation is updated and accurate

### Implementation Notes

- Include a "Troubleshooting" section for common Node version issues

### Files Likely Impacted

- `README.md`
- `ARCHITECTURE.md`
- `.cursorrules`

### Verification Commands

- Manual review

---

## Recommended Execution Order

1. **SUD-1 (Scaffold)**: Must be first to establish the environment
2. **SUD-2 (Parser)**: Essential for all other logic
3. **SUD-3 (Solver) & SUD-4 (Generator)**: Can be parallelized. Note: Generator usually depends on the Solver to verify uniqueness
4. **SUD-5 (CLI Integration)**: Depends on 2, 3, and 4
5. **SUD-6 (Docs)**: Final touch