---
name: project-release
description: Generic guarded release workflow for any Git project. Use when the user asks to publish, release, commit and push, tag and push, or uses triggers including “发布”, “来人”, “提交推送”, “打包推送”, “打标推送”, “release”, or “commit push”.
---

# Project Release

Use this skill for release closeout in any Git repository. Keep the workflow repository-local: never hard-code a project path, branch, remote, changelog path, or test command when the current repository can provide it.

## Configuration

Start from the current working directory and find the Git root with:

```bash
git rev-parse --show-toplevel
```

Before planning release commands, read `.release.yaml` from the Git root if it exists. Supported fields:

```yaml
project_name: MyProject
default_branch: main
test_commands:
  - npm test
  - npm run build
changelog_file: CHANGELOG.md
tag_prefix: v
push_remote: origin
push_branch: main
```

If `.release.yaml` is absent, use this safe default:
- Determine the current branch with `git branch --show-current`.
- Use `origin` as the push remote only if `git remote -v` shows it.
- Use `CHANGELOG.md` only if it exists; otherwise ask before creating one.
- Do not guess release tests blindly. Inspect obvious project files such as `package.json`, `pyproject.toml`, `pytest.ini`, `Cargo.toml`, `go.mod`, or README scripts, then propose test/build commands and get user confirmation before running release-impacting steps.
- Do not tag unless the user requests a tag or the repo config clearly defines tag behavior.

## Required Workflow

1. Inspect repository state first:

```bash
git status --short
git diff --stat
git ls-files --others --exclude-standard
```

2. Identify sensitive or generated files before staging. Never stage, commit, print, or copy:
- `.env`, `.env.*`
- `.venv`, `venv`, virtualenv folders
- `node_modules`
- `dist`, `build`, coverage outputs, cache outputs
- `__pycache__`, `*.pyc`
- `*.db`, `*.sqlite`, `*.sqlite3`
- actual secrets, credentials, tokens, private keys, and files containing private key material

Treat names containing `secret`, `token`, `key`, or `credential` as review indicators, not proof that a file is sensitive. Inspect only safe metadata and context; do not print or copy suspected secrets. Exclude a file if its safety cannot be established.

If any such files are modified or untracked, leave them unstaged and tell the user.

3. Run every command in `.release.yaml` `test_commands` before committing. If there is no config, run only the user-confirmed test/build commands. Stop on failure and report the failing command. If a required tool, dependency, or virtual environment is missing, identify it and report how to restore it; do not silently skip the command.

4. Update the changelog before commit when `changelog_file` is configured or a changelog exists. Keep the entry concise and based only on actual changes and verification results.

5. Before staging or committing, summarize:
- changed files that will be included
- changed files that will be excluded
- test/build commands run and their results
- changelog update
- proposed commit message

6. Stage only intended safe files. Prefer explicit paths over broad `git add .`.

7. Commit only after the summary is clear. Do not commit if tests/build failed, if sensitive files would be included, or if the user has asked for review only.

8. Before pushing, check the current branch and confirm that the intended commit is on the configured push branch. Stop if they do not match; do not switch branches or push a different branch implicitly.

9. Do not create a tag unless the user explicitly requests one or the repository configuration clearly defines when to tag and how to determine the version. A `tag_prefix` alone does not define tag behavior or a version. Do not guess a version.

10. Before any push or tag action, show the exact commands that would run, using `.release.yaml` values when available:

```bash
git push <push_remote> <push_branch>
git tag -a <tag> -m "<message>"
git push <push_remote> <tag>
```

Do not push or tag unless the user has explicitly requested release publishing in this turn and the repository is ready. If credentials are required, tell the user to enter them in their own terminal. Never ask the user to paste GitHub passwords, API keys, tokens, SSH private keys, or other secrets into Codex.

## Output

Keep the final response concise:
- conclusion
- files changed or committed
- validation commands and results
- push/tag commands shown or executed
- any files deliberately excluded
