# lefthook-go-rust-typescript

One `lefthook.yml` that runs the right pre-commit checks per language in a
polyglot repo, in parallel, with no husky/Node bootstrap.

## Problem

`husky` wants `package.json`. Go and Rust repos don't have one, and adding it
for a hook runner is wrong. `lefthook` is a single Go binary with glob-scoped
commands, so each file's language determines which checks fire.

## Snippet

`lefthook.yml`:

```yaml
commit-msg:
  commands:
    commitlint:
      run: npx --no -- commitlint --edit {1}

pre-commit:
  parallel: true
  commands:
    gofmt:
      glob: "*.go"
      run: gofmt -l {staged_files} | (grep . && exit 1 || exit 0)
    govet:
      glob: "*.go"
      run: go vet ./...
    golangci-lint:
      glob: "*.go"
      run: golangci-lint run --new-from-rev=HEAD~ --timeout=2m
    cargo-fmt:
      glob: "*.rs"
      run: cargo fmt --all -- --check
    cargo-clippy:
      glob: "*.rs"
      run: cargo clippy --all-targets --all-features -- -D warnings
    tsc:
      glob: "*.{ts,tsx}"
      run: pnpm -s typecheck
    eslint:
      glob: "*.{ts,tsx,js,mjs}"
      run: pnpm -s exec eslint {staged_files}
    shellcheck:
      glob: "*.{sh,bash}"
      run: shellcheck --severity=warning {staged_files}
    shfmt:
      glob: "*.{sh,bash}"
      run: shfmt -i 2 -ci -bn -d {staged_files}

pre-push:
  commands:
    test-affected:
      run: |
        case "$(git diff --name-only origin/main...HEAD | head -1)" in
          *.go)  go test ./... ;;
          *.rs)  cargo test --locked ;;
          *.ts|*.tsx) pnpm -s test ;;
        esac
```

Install: `brew install lefthook && lefthook install`. Lives in `.git/hooks/`.

## Why

`glob:` scopes each command to the files being committed, so a one-line
Markdown edit doesn't run `cargo clippy`. `parallel: true` lets unrelated
languages run simultaneously. The single-binary install means CI and local
dev are the same tool — no "works on my machine because I have husky 8."

## When NOT to use

Don't adopt lefthook in a repo where every contributor is already on a working
husky setup with shared config and no language drift — the migration cost
exceeds the benefit. Also skip if your team insists on Git's native hooks
infrastructure without a manager; lefthook writes shell shims to `.git/hooks/`
that can surprise people who audit that directory.

## Reference

- https://github.com/evilmartians/lefthook
