# Documentation lifecycle update

Run this workflow after code changes and before opening a pull request.

## Step 1: Detect changed files

Run:

```bash
git status --short
git diff --name-only HEAD
git diff --stat HEAD
```

Identify changed source files. Exclude:
- Generated files.
- Build artifacts.
- Lock files unless they are the only changed files.
- Documentation files when determining what code behavior changed.

If no source files changed, stop and report that no documentation lifecycle action is required.

## Step 2: Capture the code diff

For each changed source file, inspect:

```bash
git diff -- <file>
```

Also read a small amount of surrounding code context from the changed file so that documentation remains accurate.

Do not analyze unrelated files unless they are needed to understand names, public interfaces, exported types, routes, commands, configuration keys, or user-visible behavior affected by the diff.

## Step 3: Locate existing documentation

Search the repository for documentation related to the changed files, using:
- File names.
- Exported class/function/type names.
- Public API route names.
- CLI command names.
- Configuration keys.
- Feature names visible in the diff.

Look in common documentation locations first:
- README.md
- docs/
- *.md
- *.mdx
- API reference files
- Architecture decision records
- Usage examples

## Step 4: Documentation gap analysis

Create a gap analysis with this structure:
```markdown

## Documentation gap analysis

### Changed source files
- `<file>`: summary of visible code change

### Existing related documentation
- `<doc file>`: relevant section

### Gaps
- Missing documentation:
- Outdated documentation:
- Inconsistent documentation:
- No action needed:

### Confidence
- High:
- Medium:
- Low:
```

Only mark a gap when there is direct evidence from the diff and surrounding code.

## Step 5: Propose documentation edits
For each gap, propose a documentation change.

Rules:
- Use the same language, tone, heading style, and formatting as the existing documentation.
- Keep changes minimal and directly tied to the diff.
- Do not add speculative examples.
- Do not add roadmap, design rationale, or implementation details unless existing documentation already uses that style.
- If no documentation file exists, create the smallest appropriate new documentation file and add it to the nearest docs index only if such an index exists.

Before applying edits, summarize the planned files to modify and ask for approval.

## Step 6: Apply approved documentation changes
After approval, edit the documentation files.

Do not modify source code.

## Step 7: Verify scope
Run:
```bash
git diff --stat
git diff --name-only
git diff
```

Verify that only appropriate documentation files were changed.

If any non-documentation files were changed, stop and ask the user whether to revert those changes.

## Step 8: Final report
Report:
- Changed source files analyzed.
- Documentation files updated or created.
- Gaps resolved.
- Gaps intentionally left unresolved because evidence was insufficient.
- Suggested PR description snippet.
