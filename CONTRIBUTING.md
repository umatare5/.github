# Contributing

These conventions hold for every exporter published under `umatare5`. Each repository carries the rest.

## Development

Install [`gotestsum`](https://github.com/gotestyourself/gotestsum), [`golangci-lint`](https://golangci-lint.run/docs/welcome/install/local/), [`pre-commit`](https://pre-commit.com/#install) and [`gitleaks`](https://github.com/gitleaks/gitleaks#installing), then run `make pre-commit-install`.

| Command                     | Description                                  |
| :-------------------------- | :------------------------------------------- |
| `make help`                 | Display available targets and requirements   |
| `make build`                | Build the binary into `./tmp`                |
| `make lint`                 | Run golangci-lint and tidy go.mod            |
| `make test-unit`            | Run unit tests with coverage using gotestsum |
| `make test-unit-coverage`   | Generate the HTML coverage report            |
| `make clean`                | Remove the build and coverage artifacts      |
| `make image`                | Build the Docker image                       |
| `make pre-commit-install`   | Install the pre-commit hooks                 |
| `make pre-commit-test`      | Run every hook across the tree               |
| `make pre-commit-uninstall` | Remove the pre-commit hooks                  |

- **Hook order** — the branch guard, `golangci-lint`, `actionlint`, `gitleaks`, then `markdownlint-cli2`.
- **The guard carries `fail_fast`** — a commit on `main` stops there, so work on a branch.
- **Only `gitleaks` comes from `PATH`** — pre-commit builds the rest at the versions it pins.
- **The markdown hook runs `--fix`** — it rewrites files, so reach it with `make pre-commit-test`.
- **`make build` skips a rebuild** — the file target does nothing while the binary exists.
- **`make clean` takes `./tmp` whole** — worktrees and fetched data go with the binary.

## Build

`make image` cross-compiles a Linux binary for the host architecture into `./tmp/image`, then builds from there, because the `Dockerfile` expects the GoReleaser layout of a binary beside `LICENSE` and `NOTICE`. The image is tagged `$USER/<name>` and declares its ports without publishing them.

Released images carry `amd64` and `arm64`, and GoReleaser pushes them to `ghcr.io/umatare5/<name>`.

## Testing

`make test-unit` runs every package under `gotestsum` with `-race` and a coverage profile, and CI runs the same against a threshold each repository sets.

- **Placement** — a test is a `*_test.go` beside the code, named for the behaviour it pins.
- **Mutation** — check a new test by reversing the change it pins and watching it fail.
- **Fixtures** — a fixture carries the shape the real source emits, not an invented one.
- **Addresses** — RFC 5737 or RFC 1918, never a monitored network or a real device.
- **Example rules** — CI lints and unit-tests them with `promtool`, which no hook covers.

## Code Style

`golangci-lint` enforces what `.golangci.yml` configures.

A comment records only what the code cannot say, and every change stays minimal.

A HELP string is one sentence stating the reading of one series. Prometheus vocabulary is used as the ecosystem uses it: a `--collector.<name>` flag switches a collector, and a named series group is a family.

## Documentation

Every fact has one page that owns it, and the other pages link to it rather than restating it. `README.md` says what the exporter is and how to run it, and the pages under `docs/` carry the metric catalogue, the flag reference and the rules every collector obeys.

- **Headings are pinned** — `.markdownlint-cli2.jsonc` fixes each page's `#` and `##` in order.
- **Contracts travel** — a heading change ships with its contract in the same pull request.
- **The transcript is verbatim** — `docs/help.md` carries the binary's own `--help` output.
- **Links are checked in CI only** — that run reaches third-party hosts, and `lychee .` reproduces it.

## Release

1. Rename `## [Unreleased]` in `CHANGELOG.md` to `## [vX.Y.Z]` and add that version's link at the foot.
2. Update the version in the `VERSION` file.
3. Update the `VERSION:` line in the `--help` transcript in `docs/help.md`.

Merging one pull request with all three files starts the release. A push to `main` touching `VERSION` tags the commit, pushes the container images and uploads the artifacts to a draft release.

- **The images go public before the draft** — discarding the draft withdraws none of them.
- **A prerelease tag pushes no image** — its draft carries the archives alone.
- **The release links 404 until the merge** — `lychee.toml` excludes the tag and compare patterns.
- **There is no manual trigger** — the push runs it, and a weekly snapshot build tags nothing.

## Pull Requests

1. Fork the repository and create a feature branch.
2. Commit with a [Conventional Commits](https://www.conventionalcommits.org/) subject and a `Signed-off-by:` trailer.
3. Record the change under `## [Unreleased]` in `CHANGELOG.md`, rebase against `main`, then open the PR.

Nothing in a commit identifies a monitored system or carries a credential.

- **`gitleaks` reads shapes** — its rules catch a token or a key, so addresses are your own care.
- **Captures live under `tmp/`** — git, the Docker context, the linter and `air` all ignore it.
- **Credentials stay in the environment** — `.env` and `.envrc` are git-ignored, so they belong there.
- **Scanners run weekly** — `govulncheck` and CodeQL, on top of every push.
- **Advisories need a path** — name the call path from `./cmd` that reaches the finding.
