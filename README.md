# EpicArena Infrastructure

> Infrastructure templates and deployment configurations.

## Contents

- Agent configuration templates (desensitized)
- Deployment scripts
- GitHub Actions workflows (auto-audit SOP)
- Server configuration references

## Auto Audit SOP

This repository contains GitHub Actions workflows for automatic code review:

| Workflow | Trigger | Purpose |
|----------|---------|---------|
| `secret-scanning.yml` | PR / Push | Detect sensitive files and hardcoded secrets |
| `config-audit.yml` | PR | Audit config files for real credentials |
| `core-leak-check.yml` | PR | Prevent core code from leaking to public repos |
| `skill-security-audit.yml` | PR (skills/) | Security audit for open-source skills |

## Note

Configuration files in this repo are **templates only**. 
Real credentials are managed via environment variables and never committed.

## License

MIT

## POS Test

Epic SOP integration test - 2026-05-06.
