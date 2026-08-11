# Dev Logbook

Personal private engineering logbook for searchable work history, reusable fixes, and interview preparation.

## Purpose

Use this repo to record meaningful bugs, fixes, deployments, investigations, features, and lessons. The goal is to make real work experience credible, useful, and easy to find later.

## Layout

```text
dev-logbook/
├── AGENTS.md
├── README.md
├── templates/
│   └── log-entry-template.md
└── logs/
    └── YYYY/
        └── MM/
            └── YYYY-MM-DD_short-title.md
```

Use one file per meaningful issue or task:

```text
logs/2026/08/2026-08-11_php-version-500-error.md
logs/2026/08/2026-08-11_queue-jobs-not-running.md
logs/2026/08/2026-08-11_datatable-pagination.md
```

## Daily Workflow

At the end of the day, paste rough notes into Codex:

```text
Add today's work to dev-logbook. Create separate issue-wise files for each meaningful item.
```

Codex should split the notes into separate files, keep each entry searchable, and include the resume/interview bullet inside the same file.

## Entry Rules

- Keep one Markdown file per meaningful item.
- Use readable filenames.
- Include exact commands where useful.
- Include verification, lesson, impact, tags, and resume/interview bullet.
- Skip tiny noise that will not help future debugging or interviews.
- Never include real customer data, secrets, tokens, credentials, or private order data.

## Search Examples

```bash
git grep "aaPanel"
git grep "Laravel"
git grep "500-error"
git grep "#deploy"
git grep "queue" logs/2026/08
```

## Git Safety

This repo must use the personal GitHub account `spk0626`, not the Sarasavi IT account.

Before pushing:

```bash
gh auth status
git config user.name
git config user.email
git remote -v
```

Expected:

```text
spk0626
spk0626@users.noreply.github.com
https://github.com/spk0626/dev-logbook.git
```
