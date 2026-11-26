# Copilot repository guide

This is the Next.js framework monorepo (about 29,000 tracked files), not a
single Next.js application. It contains TypeScript/JavaScript packages and
Rust-based SWC and Turbopack tooling. Next.js builds full-stack React apps
for Node.js and edge runtimes. Trust these instructions and `AGENTS.md`;
search further only when they are incomplete or appear incorrect.

## Setup and validation

Run commands from the repository root. Use Node.js >=20.9 (the `.node-version`
is 20; CI primarily uses 22 and also tests 20) and pnpm 10.33.0 from
`package.json`. Enable a pnpm executable on `PATH` with `corepack enable pnpm`
before installing. If the global shim directory is not writable, use
`mkdir -p /tmp/pnpm-bin && corepack enable pnpm --install-directory /tmp/pnpm-bin`
and prepend `/tmp/pnpm-bin` to `PATH`. Calling `corepack pnpm install` without
a pnpm executable fails in the SWC postinstall (`spawn pnpm ENOENT`).

```sh
pnpm install --frozen-lockfile
pnpm build
```

Always install before building: a clean checkout otherwise fails with
`turbo: not found`. Installation runs native SWC setup and needs registry
access. On Linux with Node 22, a fresh build took about 103 seconds after
installation. For Rust changes, install Rust through rustup, a C compiler, and the
toolchain in `rust-toolchain.toml` (nightly-2026-05-15); use `pnpm build-all`
to build JavaScript and native code, or `pnpm swc-build-native` for bindings.
See `contributing/core/developing.md` and `contributing/core/building.md`.

After bootstrap, `pnpm --filter=next dev` watches the framework build; stop it
before a full build to avoid build corruption. For a local app, use the built
CLI via `pnpm next dev <app-directory>`; see
`contributing/core/developing-using-local-app.md`.

Build before integration tests and rebuild after source changes. Run focused
tests in the relevant mode and bundler (the unsuffixed `test-dev` and
`test-start` scripts use webpack):

```sh
pnpm exec jest --runInBand test/unit/htmlescape.test.ts
pnpm test-dev-turbo test/path/to/test.test.ts
pnpm test-dev-webpack test/path/to/test.test.ts
pnpm test-start-turbo test/path/to/test.test.ts
pnpm test-start-webpack test/path/to/test.test.ts
```

Browser tests need Playwright browsers and system dependencies; CI uses
`pnpm playwright install --with-deps <browser>`. E2E tests generally create
isolated installations in a temporary directory. `NEXT_SKIP_ISOLATE=1` can
speed local debugging but is not compatible with every test and does not
validate normal package resolution. See `contributing/core/testing.md`.
`pnpm test-unit --runInBand` runs the broader unit suite.

`pnpm lint` runs type, Prettier, ESLint, AST-grep, language and Turbo checks;
`pnpm lint-fix` formats and fixes lint issues. `pnpm --filter=next types`
checks/builds Next.js declarations. The pre-commit hook runs `pnpm lint-staged`:
Prettier for supported files, ESLint for JS/TS/MDX, and rustfmt for Rust.
CI's `.github/workflows/build_and_test.yml` and `build_reusable.yml` install,
build and run separate lint, type, unit, Rust and integration jobs; reproduce
the failing job's mode, bundler and environment. Docs guidance is in
`contributing/docs/adding-documentation.md`.

## Where to work

- `packages/next/src/`: framework source; `cli/next-{dev,build,start}.ts` are
  CLI entry points, `server/` serves requests, `client/` handles browser
  runtime, and `build/` implements compilation. Output is `packages/next/dist/`.
- `packages/`: other published packages including `create-next-app`,
  `eslint-plugin-next`, `font` and `third-parties`.
- `crates/`: Next.js Rust/SWC crates; `turbopack/`: Turbopack Rust subtree.
- `test/unit/`, `test/development/`, `test/production/`, `test/e2e/`:
  corresponding suites; shared test utilities are in `test/lib/`. Use
  `pnpm new-test -- --args <true|false> <name> <e2e|production|development|unit>`
  for new suites, and prefer extending existing relevant suites.
- `docs/`, `errors/`, `examples/`, `bench/`: documentation, error pages,
  example apps and benchmarks. Root `readme.md` links to
  `packages/next/README.md`; `contributing.md` indexes contributor guides.
- Root `package.json`, `pnpm-workspace.yaml`, `turbo.json`, `Cargo.toml`,
  `jest.config.js`, `tsconfig.json`, `eslint.config.mjs`, `.prettierrc.json`,
  `.prettierignore` and `.config/ast-grep/` define scripts and tooling.
  `scripts/` contains build/test helpers; `.github/workflows/` contains CI.

Read applicable `README.md` files along the path before editing a directory.
`AGENTS.md` provides more detailed conventions, mode-specific test commands,
feature-flag and runtime guidance; `CLAUDE.md` is its symlink.
