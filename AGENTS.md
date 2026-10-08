# Repository guidance for AI agents

## Purpose and layout

This repository contains tested Bash VPS startup scripts stored as `.txt` files, plus verification command bundles and documentation. It is not an application with a build system.

- `UserData-DigiOcean.txt`: combined WireGuard/Teleproxy setup using DigitalOcean metadata.
- `UserData-generic.txt`: combined setup with public-IP detection fallbacks.
- `verification-DigiOcean.txt` and `verification-generic.txt`: combined verification command bundles.
- `WireGuard-Teleproxy-VPS-Scripts/UserData-wireguard.txt` and `UserData-teleproxy.txt`: standalone installers.
- `WireGuard-Teleproxy-VPS-Scripts/verification-wireguard.txt` and `verification-teleproxy.txt`: standalone verification command bundles.
- `README.md`: primary public documentation.
- `ReadMe.txt`: optional local legacy quick reference, excluded from Git.

## Editing expectations

- Preserve the maintainer-tested script behavior unless the task requests a behavior change. Documentation work should leave installer and verification files unchanged.
- Read the relevant installer and verifier before editing. Shared logic is duplicated across variants; assess related variants and keep intentional provider/service differences.
- Keep `.txt` filenames and copy/paste user-data workflows unless a rename or restructuring is requested.
- Keep startup scripts compatible with Bash on Debian/Ubuntu servers using apt-get and systemd. Teleproxy downloads currently target amd64.
- If changing ports, peer counts, subnets, paths, or output formats, review corresponding verification checks and documentation for consistency.
- Verification files wrap a script in a quoted heredoc, then chmod and execute it. Preserve both the wrapper and generated script behavior.
- Do not introduce unrelated refactors, dependencies, or generated planning documents for routine edits.
- Document supported assumptions separately from tested environments. Do not invent OS versions, deployment results, or guarantees.

## Safe validation

Do not execute startup scripts on the development machine. They install packages, regenerate credentials, overwrite `/etc` configuration, and enable services. Do not execute verification bundles locally either: their wrappers create and run scripts intended for the VPS.

For a syntax-only check of all eight command files, run from the repository root:

```bash
for file in UserData-*.txt verification-*.txt WireGuard-Teleproxy-VPS-Scripts/*.txt; do
  bash -n "$file" || exit 1
done
```

`bash -n` checks wrapper syntax but treats heredoc contents as text. For verification changes, also extract the embedded verifier without executing it and check that body with `bash -n`. Review generated systemd and WireGuard configuration separately; shell parsing does not validate their contents.

For documentation changes, check relative links, file paths, command examples, and consistency with the scripts. Behavior changes require a fresh, disposable VPS in an explicitly authorized environment. Report exactly which checks ran; never present syntax checks as a successful deployment.

## Credentials and external actions

- Never commit generated WireGuard keys, client profiles, QR codes, Teleproxy secrets, environment files, or server summary files.
- Redact credentials from logs and examples. Use placeholders for server IPs and secrets in documentation.
- Creating VPS resources, connecting to servers, or publishing changes requires authorization within the current task. A request to update local documentation does not authorize those actions.
- Installers are for initial provisioning and are not idempotent: rerunning them replaces credentials/configuration. Account for that before suggesting commands on an existing server.
