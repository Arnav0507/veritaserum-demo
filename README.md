# Veritaserum demo

This tiny Go project shows how [Veritaserum](https://github.com/Arnav0507/Veritaserum)
ties `AGENTS.md` conventions to machine-checkable claims and CI drift detection.

## What the context says

`AGENTS.md` states four conventions:

- JSON marshaling uses `base.JSONMarshal`, never `encoding/json` directly
- Logging uses `base.Infof`, never `fmt.Printf`
- New UI file names use kebab-case
- The default public port is `:4984`

Each convention maps to a typed claim in `claims/demo.yml`.

## Try it locally

```bash
python -m pip install "git+https://github.com/Arnav0507/Veritaserum@v0.3.0"
veritaserum check --repo .
```

Exit code `0` — the code matches the instructions.

## Introduce drift

Apply the bundled patch to break the logging convention:

```bash
patch -p1 < drift.patch
veritaserum check --repo .
```

Exit code `1` — Veritaserum reports `demo-no-printf` with evidence in `main.go`
and anchors the SARIF result at `AGENTS.md#L6`.

Check only claims touched by your changes:

```bash
git stash -u  # optional: save drift
git commit -am "introduce drift"  # after applying patch
veritaserum affected --repo . --base HEAD~1 --head HEAD
```

## GitHub Actions

Pull requests run [Veritaserum](https://github.com/Arnav0507/Veritaserum) via
`.github/workflows/veritaserum.yml`:

1. **`check`** — full claim verification + SARIF upload
2. **`affected`** — re-check only claims touched by the PR diff and flag stale context

When a change introduces `fmt.Printf`, both steps fail on [PR #1](https://github.com/Arnav0507/veritaserum-demo/pull/1).

## Next steps

- Draft additional claims with `veritaserum suggest --repo . --context AGENTS.md`
- Review generated YAML before committing — verification stays deterministic
- Commit a baseline after accepting known brownfield drift
