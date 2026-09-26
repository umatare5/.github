# Contributing

These conventions apply to every Prometheus exporter under **umatare5**. Each repository's own `CONTRIBUTING.md` covers the rest.

## Development

Install [`gotestsum`][gotestsum], [`golangci-lint`][golangci-lint], [`pre-commit`][pre-commit] and [`gitleaks`][gitleaks], then run `make pre-commit-install`.

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

The pre-commit hooks work as follows.

- **Hook order** – the branch guard, `golangci-lint`, `actionlint`, `gitleaks`, then `markdownlint-cli2`.
- **Branch guard** – it sets `fail_fast`, so a commit on `main` stops there. Work on a branch.
- **Tool versions** – only `gitleaks` comes from `PATH`, and pre-commit builds the rest at pinned versions.
- **Markdown hook** – it runs `--fix` and rewrites files, so run it with `make pre-commit-test`.

## Build

`make image` cross-compiles a Linux binary for the host architecture into `./tmp/image` and builds the image from there. It uses that directory because the `Dockerfile` expects the GoReleaser layout of a binary next to `LICENSE` and `NOTICE`. The image is tagged `$USER/<name>` and declares its ports without publishing them.

Released images are built for `amd64` and `arm64`, and GoReleaser pushes them to `ghcr.io/umatare5/<name>`.

## Testing

`make test-unit` runs every package under `gotestsum` with `-race` and a coverage profile. CI runs the same tests and checks the coverage against a threshold that each repository sets.

Tests follow these conventions.

- **Placement** – a test lives in `*_test.go` next to the code and is named for the behavior it covers.
- **Mutation** – you check a new test by reversing the change it covers and watching the test fail.
- **Fixtures** – a fixture carries the shape the real source emits, not an invented one.
- **Addresses** – tests use [RFC 5737][rfc5737] or [RFC 1918][rfc1918], never a monitored network or a real device.
- **Example rules** – CI lints and unit-tests them with `promtool`, and no hook does.

## Code Style

`golangci-lint` enforces the rules in `.golangci.yml`, and the code also follows these conventions.

- **Comments** – a comment records only what the code cannot say.
- **Changes** – every change stays minimal.
- **HELP strings** – each one is a single sentence that says what the value of one series means.
- **Terms** – Prometheus terms keep their standard meaning.
- **Collector** – the unit that a `--collector.<name>` flag turns on or off.
- **Family** – the series that share one name.

## Documentation

Every fact has one page that owns it, and the other pages link to it rather than repeating it. `README.md` says what the exporter is and how to run it. The pages under `docs/` carry the metric catalog, the flag reference and the rules every collector follows.

Every page follows these rules.

- **Headings** – the `MD043` contract in `.markdownlint-cli2.jsonc` pins each page's `#` and `##` in order.
- **Contracts** – a heading change updates its contract in the same pull request.
- **Transcript** – `docs/help.md` carries the binary's own `--help` output verbatim.
- **Links** – only CI checks them, because the check reaches third-party hosts. `lychee .` runs it locally.

## Release

A release changes three files in one pull request.

1. Rename `## [Unreleased]` in `CHANGELOG.md` to `## [vX.Y.Z]` and add that version's link at the bottom.
2. Update the version in the `VERSION` file.
3. Update the `VERSION:` line in the `--help` transcript in `docs/help.md`.

Merging that pull request starts the release. A push to `main` that changes `VERSION` tags the commit, pushes the container images and uploads the artifacts to a draft release.

- **Trigger** – only the push starts a release, and the weekly snapshot build creates no tag.
- **Images** – they go public before the draft, so deleting the draft removes none of them.
- **Prereleases** – a prerelease tag pushes no image, and its draft holds only the archives.
- **Release links** – they return 404 until the merge, so `lychee.toml` excludes the tag and compare URLs.

## Pull Requests

Open a pull request in three steps.

1. Fork the repository and create a feature branch.
2. Commit with a [Conventional Commits][conventional-commits] subject and a `Signed-off-by:` trailer.
3. Record the change under `## [Unreleased]` in `CHANGELOG.md`, then rebase onto `main` and open the PR.

Nothing in a commit identifies a monitored system or carries a credential.

- **Captures** – they live under `tmp/`, which git, the Docker context, the linter and `air` all ignore.
- **Credentials** – they stay in the environment or in `.env` and `.envrc`, which git ignores.
- **Scanning** – `gitleaks` matches the patterns of a token or a key, so checking addresses is up to you.

CI also scans the code.

- **Static analysis** – `govulncheck` and CodeQL run every week in addition to their push runs.
- **Advisories** – an advisory needs the call path from `./cmd` that reaches the finding.

[gotestsum]: https://github.com/gotestyourself/gotestsum
[golangci-lint]: https://golangci-lint.run/docs/welcome/install/local/
[pre-commit]: https://pre-commit.com/#install
[gitleaks]: https://github.com/gitleaks/gitleaks#installing
[rfc5737]: https://datatracker.ietf.org/doc/html/rfc5737
[rfc1918]: https://datatracker.ietf.org/doc/html/rfc1918
[conventional-commits]: https://www.conventionalcommits.org/
