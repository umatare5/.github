# Security Policy

This policy covers every exporter published under `umatare5`. Each repository carries the rest.

## Supported Versions

Only the latest release carries fixes, and no older tag gets a patch branch. Reproduce a finding against that release before reporting it.

## Reporting a Vulnerability

Report privately through GitHub Security Advisories, never through an issue or a pull request. Open the repository's **Security** tab and choose **Report a vulnerability**.

The response is best effort, with no promised window. The advisory goes out once the fix ships, carries a CVE request, and credits the reporter unless they ask otherwise.

## What to Include

**Redact these first.** Neither belongs in a report.

- A credential, from a flag, an environment variable, a header or a log line
- An address, a hostname or an account identifier of a monitored system

Then include the following.

- **Affected versions** — the release you reproduced against, and the image tag if any.
- **Reproduction steps** — the flags and environment variables in force, and what it was reading.
- **Output** — the `/metrics` body or the log lines, with every value above removed.
- **Impact** — state the exploit scenario, and what it reaches.
- **Suggested fix** — propose a remediation where you have one; this one is optional.
- **Disclosure status** — say whether it is shared elsewhere, and give your plan for sharing it.

## Exposure

- **Metrics** — `/metrics` and the landing page serve unauthenticated plain HTTP.
- **Posture** — that is documented rather than accidental, so keep the port on a controlled path.
- **Container** — the image is built from `scratch`, runs as UID 65534 and carries one CA bundle.
- **The published image** — a defect in what `ghcr.io` serves is reportable, not only one in the source.

## Out of Scope

- A defect in the system an exporter reads belongs to that system's vendor.
- A dependency advisory with no path reachable from `./cmd`, unless you show the reachable path.
- An operator's own configuration, which each repository's own flag reference covers.
