# Scripts

PowerShell scripts used to build and configure the lab will live here.

Planned:

- `Build-LabAD.ps1`: creates the OU structure, users, security groups and password/lockout policy. It's **idempotent**: it checks whether each object already exists before creating it, so it can be rerun safely without errors or duplicates.

Rules for anything committed here:

- No real or lab passwords in scripts. Prompt for them at runtime (`Read-Host -AsSecureString`) instead.
- Comment each section so the script explains *why*, not just *what*.
