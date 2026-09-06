# Security Policy

This repository contains a `cloud-init`/`autoinstall` configuration for
unattended Ubuntu Server installs, not a versioned software package, so there
are no "supported versions" to track — the policy below always applies to the
current `main` branch.

## Reporting a Vulnerability

If you find a security issue with this configuration (for example, an
`autoinstall` setting that would leave an installed system insecure by
default, or a problem in the `late-commands` provisioning script), please
report it privately using GitHub's
[private vulnerability reporting](https://docs.github.com/en/code-security/security-advisories/guidance-on-reporting-and-writing/privately-reporting-a-security-vulnerability)
feature on this repo's **Security** tab, rather than opening a public issue.

You should get an initial response within a few days. Once a fix is
available, it will be merged into `main` and noted in the commit history.

## Note on Included Secrets

`user-data` in this repository is a template: the `identity.password` field
is a placeholder (`$6$CYOURPASSWORD_HASH`), not a real password hash. If you
fork or reuse this config, generate your own hash with `openssl passwd -6`
and never commit a real password hash or other credentials to a public repo.
