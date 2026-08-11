# Dev Logbook Agent Instructions

This is a personal private dev logbook repo owned by `spk0626`.

## Account Safety

- Always use the personal GitHub account `spk0626` for this repo.
- Never commit or push this repo using `Sarasavi-IT`, `sarasavi-it`, or `dinithi.w@sarasavi.org`.
- Before any push, verify:

```bash
gh auth status
git config user.name
git config user.email
git remote -v
```

Expected values:

- Active GitHub account: `spk0626`
- Git user: `spk0626`
- Git email: `spk0626@users.noreply.github.com`
- Remote: `https://github.com/spk0626/dev-logbook.git`

## Logbook Rules

- Default to one Markdown file per meaningful issue, fix, task, deployment, investigation, or learning.
- When given a full day of notes, split them into separate issue-wise files.
- Store entries under `logs/YYYY/MM/YYYY-MM-DD_short-title.md`.
- Keep the resume/interview bullet inside the same issue file.
- Do not create daily, weekly, or separate resume summary files unless explicitly requested.
- Do not log tiny noise. Log things worth searching later.

## Privacy Rules

- Do not include customer emails, phone numbers, credentials, tokens, private order data, secrets, or sensitive production data.
- Sanitize internal URLs, IDs, screenshots, logs, and examples before writing them.
- Keep commands copy-paste ready, but replace secrets and private values with clear placeholders.

## Entry Quality

Each entry should include:

- problem
- context or reproduction
- what was tried
- root cause
- fix or commands
- verification
- lesson
- impact
- tags
- resume/interview bullet

Keep entries clear, short, and useful for future debugging.

## Writing Style

- Write like a developer leaving notes for their future self.
- Use plain, direct language. Prefer exact errors, commands, paths, files, routes, and observed behavior.
- Avoid AI-sounding filler such as "seamlessly", "robust", "leveraged", "comprehensive", "streamlined", "mission-critical", and vague praise.
- Do not inflate impact. If there is no measured number, say the real impact plainly.
- Keep technical sections practical, not polished for marketing.
- Only the `Resume / Interview Bullet` section should sound resume-ready.
- Keep paragraphs short. Use bullets for symptoms, causes, fixes, and checks when that is easier to scan.
- Preserve uncertainty. If the root cause was suspected but not fully proven, say that.
- Do not rewrite messy real debugging into a fake perfect story; keep the useful trail of what was tried.
