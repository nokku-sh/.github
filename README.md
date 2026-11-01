# Nokku shared config

Reusable workflows, a renovate preset and config templates for all nokku-sh repos.

## Workflows

| Workflow          | What it does                                                                                                                                                                                                                |
| ----------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `go-ci.yaml`      | `go mod tidy -diff`, golangci-lint, `go test -race -shuffle=on`, govulncheck, `goreleaser check`, actionlint                                                                                                                |
| `go-release.yaml` | Computes the next version with git-cliff, runs `go-ci` on that commit, tags, then releases with goreleaser or a plain GitHub release for libraries. Pushes deb, rpm and apk packages to Cloudsmith. Removes untagged images |
| `go-nightly.yaml` | On every push to main, builds the `kos` image with ko, pushes `:nightly`, signs it and removes the previous one                                                                                                             |
| `actionlint.yaml` | Lints workflows, for repos that are not Go                                                                                                                                                                                  |

Callers live in `templates/go/workflows/`, copy them into a new repo. Extra jobs go next to the shared one:

```yaml
jobs:
  ci:
    uses: nokku-sh/.github/.github/workflows/go-ci.yaml@main
```

Callers pin `@main` on purpose so a fix lands everywhere at once. Everything inside this repo is SHA pinned and renovate keeps it fresh.

`actions/go-cache` goes right after setup-go with `cache: false`, in go-ci and in any extra job of a caller. Self-hosted runners keep their own Go cache. Hosted runners only save it from `main`, other branches restore from it. Release and nightly builds start from empty Go caches on any runner.

Conventions the workflows rely on:

- `go.mod` at the root, `.goreleaser.yaml` (not `.yml`) if the repo ships binaries
- `.github/actions/prepare/action.yml` in the calling repo runs right after Go is set up, in CI, nightly and release. It builds frontends, sets cgo flags or installs test tools. `RUN_TESTS=true` is only set in CI, use it to start services the tests need
- `nfpms:` in the goreleaser config turns on the Cloudsmith push to `nokku/<package name>`, one Cloudsmith repository per package, and makes the `CLOUDSMITH_API_KEY` secret mandatory
- `kos:` in the goreleaser config turns on the ghcr login and image cleanup, the nightly reads `main`, `base_image`, `repositories`, `platforms` and `sbom` from the first entry
- the license public key is passed as the `NOKKU_LICENSE_PUBLIC_KEY` secret and read as `.Env.NOKKU_LICENSE_PUBLIC_KEY`
- `runner: macos-latest` moves the release job to a Mac, for repos with darwin cgo builds. It wins over `RUNNER`
- a `RUNNER` variable on a repo or the org moves every job to that runner label, `self-hosted` for the home runners. Fork PRs always stay on hosted runners

## Releases

Releases are manual for now, run the Release workflow or push a `v*` tag. Nothing is released unless git-cliff finds commits that bump the version. Commits that only touch docs, workflows, tests or dev config don't bump it, and neither do `chore(deps)` updates. The list lives in `GIT_CLIFF_EXCLUDE_PATH` in `go-release.yaml`.

The caller template has two commented triggers. The schedule gives weekly releases. The push to main trigger releases right away when a `sec` commit lands. Neither one cuts a repo's first release, that is always done by hand. Stable image tags are never deleted, only untagged leftovers and the previous nightly.

## Verifying a release

The signing job lives in this repo, so every release carries the same certificate identity. The repository flag is what ties a release to its repo, always pass it. nokku releases the core, nk and nokkud together under one `checksums.txt`:

```sh
cosign verify-blob --bundle checksums.txt.sigstore.json \
  --certificate-identity https://github.com/nokku-sh/.github/.github/workflows/go-release.yaml@refs/heads/main \
  --certificate-oidc-issuer https://token.actions.githubusercontent.com \
  --certificate-github-workflow-repository nokku-sh/nokku \
  checksums.txt
```

Renaming or moving `go-release.yaml` changes that identity and breaks every `install.sh`.

## Renovate

Each repo's `.github/renovate.json` only extends `local>nokku-sh/.github`, the rules live in `default.json`. Updates wait 7 days after their release. Everything below a major automerges with CI as the gate, majors wait for a review. Updates under `web/` or `core/web/` are committed as `fix(deps)` because they ship in the binary, vulnerability fixes as `sec(deps)`.

## Templates

`templates/` holds files that can't be referenced remotely. Run `./sync.sh` to copy them into the sibling checkouts, `./sync.sh --check` to only report drift. nokku extends the template in its own `.golangci.yml`.
