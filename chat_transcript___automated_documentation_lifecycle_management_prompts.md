# Chat Transcript - Automated Documentation Lifecycle Management Prompts

## User

I want to create a prompt, or collections of prompts for Cline which solve the "Requirements — Automated Documentation Lifecycle Management" listed below. Create three different solutions and explain the weakneses and strnghts of each solution in a summary after all the three solutions. See what Cline can do here: <https://docs.cline.bot/customization/overview> .

# Requirements — Automated Documentation Lifecycle Management

## Functional Requirements

FR-01 — Code Change Detection The system shall monitor version control (e.g. Git) for commits, merges, and pull requests, and identify which source files have changed.

FR-02 — Documentation Gap Analysis The system shall compare changed code against existing documentation to detect missing, outdated, or inconsistent documentation entries.

FR-03 — Automated Documentation Generation The system shall generate draft documentation updates for detected gaps using an LLM, referencing the actual code diff and surrounding context.

FR-04 — Documentation Publishing Upon approval, the system shall make the appropriate documentation changes.

FR-05 — Missing Documentation Creation The system shall create missing documentation, based on the current code changes, where none currently exists.

## Non-Functional Requirements

NFR-01 — Accuracy Generated documentation drafts shall be factually grounded in the actual code; the LLM shall not make assumptions.

NFR-02 — Output Scope Limitation The LLM shall only generate content directly related to the code diff provided; it shall not infer or document functionality not present in the changed files.

NFR-03 — Documentation Consistency Generated documentation shall use the same format, language, and style as existing documentation in the repository.

## Integration Requirements

IR-01 — Git integration shall be used for change detection.

IR-02 — LLM via Cline shall be used for gap analysis and draft generation.

## PoC Scope Constraints

SC-01 — The PoC shall target a single repository.

SC-02 — If a script is created as part of the PoC, that script shall be implemented in Python.

## Assumptions

A-01 — All developers have their own local agent.

A-02 — Developers have read/write access to the Git repository and can open pull requests.

A-03 — Existing documentation, where present, is stored in the same Git repository as the source code.

A-04 — The LLM will not be used to make decisions about code correctness, only documentation coverage and content.

## Assistant

Below are three alternative Cline prompt/customization solutions for a PoC that automates documentation lifecycle management for a single repository.

I’m assuming the repository already contains source code and documentation, and that developers run Cline locally with read/write Git access. I’m also assuming the PoC should produce draft documentation changes in the same repo, not publish to an external docs platform.

Cline is a good fit for this because it supports project-level customizations through Rules, Skills, Workflows, Hooks, and `.clineignore`; Workflows can define repeatable multi-step processes; Hooks can inject deterministic validation or guardrail logic; and the Cline CLI can be used from the terminal or automation contexts.

## Solution 1: Manual developer-triggered Cline workflow

**Concept.**\
This solution uses a Cline Workflow that developers run locally after completing code changes and before opening a pull request. It is the simplest PoC: Git provides change detection, Cline performs documentation gap analysis, and Cline edits or creates documentation files after developer approval.

Cline Workflows are Markdown files that define repeatable task steps and can be invoked by typing `/` followed by the workflow filename, such as `/release-prep.md`. Workspace workflows live in `.clinerules/workflows/`, which makes them versionable with the repository.

### Repository files

```text
repo/
├── .clinerules/
│   ├── documentation-lifecycle.md
│   └── workflows/
│       └── docs-lifecycle.md
├── .clineignore
├── docs/
└── src/
```

### Prompt 1: `.clinerules/documentation-lifecycle.md`

```markdown
# Documentation lifecycle rules

You are responsible for documentation coverage only. Do not judge code correctness.

## Scope

When asked to update documentation after code changes:

- Use Git as the source of truth for changed files.
- Analyze only changed source files and directly related documentation.
- Do not document behavior that is not present in the changed code.
- Do not infer hidden business rules, side effects, integrations, or runtime behavior unless directly visible in the diff or surrounding code.
- If the diff is insufficient to document a behavior confidently, write a reviewer note instead of inventing content.

## Grounding requirements

Every generated documentation change must be grounded in:

- The Git diff.
- The surrounding code context.
- Existing documentation style and structure.

If you cannot find enough evidence, use this format:

> Documentation reviewer note: I could not verify [specific point] from the provided code diff/context.

## Style consistency

Before editing documentation:

- Inspect existing documentation near the target section.
- Preserve heading style, tone, terminology, examples, and formatting.
- Prefer modifying existing documentation over creating new documentation.
- Create new documentation only when no suitable existing location exists.

## Output limitations

Only modify documentation files, examples, comments intended as documentation, or docs index files.

Do not modify application source code, tests, package configuration, CI configuration, or generated files unless explicitly instructed by the user.
```

### Prompt 2: `.clinerules/workflows/docs-lifecycle.md`

```markdown
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

- `README.md`
- `docs/`
- `*.md`
- `*.mdx`
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

```

### Optional `.clineignore`

```gitignore
# Dependencies
node_modules/
.venv/
vendor/

# Build outputs
dist/
build/
out/
.next/

# Coverage
coverage/

# Secrets
.env
.env.*

# Large/generated artifacts
*.min.js
*.map
*.sqlite
*.parquet
```

The `.clineignore` file controls which files Cline can access and can reduce irrelevant context by excluding dependencies, build outputs, generated files, large files, and secrets while keeping source code, docs, config, and tests accessible.

### How this maps to the requirements

| Requirement | Coverage |
| --- | --- |
| FR-01 | Workflow runs `git status`, `git diff --name-only`, and `git diff`. |
| FR-02 | Cline searches docs and compares them with changed code. |
| FR-03 | Cline drafts grounded documentation edits. |
| FR-04 | Cline applies edits only after approval. |
| FR-05 | Workflow instructs Cline to create missing docs when no suitable file exists. |
| NFR-01 | Rule forbids unsupported assumptions. |
| NFR-02 | Rule limits output to diff-related behavior. |
| NFR-03 | Rule requires matching existing docs style. |
| IR-01 | Uses Git CLI. |
| IR-02 | Uses Cline for analysis and generation. |
| SC-01 | Runs in one repo. |
| SC-02 | No script required. |

## Solution 2: Python-assisted workflow with structured evidence pack

**Concept.**\
This solution adds a deterministic Python script that gathers Git changes, relevant diffs, candidate documentation files, and metadata into a structured evidence pack. Cline then uses that evidence pack to perform gap analysis and generate documentation updates.

This is stronger than Solution 1 for accuracy because the script narrows the input and makes the evidence explicit. It also satisfies the PoC constraint that any created script must be Python.

Cline Workflows can reference command-line tools installed locally, including Git, Python, and other CLI tools.

### Repository files

```text
repo/
├── .clinerules/
│   ├── documentation-lifecycle.md
│   └── workflows/
│       └── docs-lifecycle-evidence-pack.md
├── tools/
│   └── docs_lifecycle_pack.py
├── .clineignore
├── docs/
└── src/
```

### Prompt 1: `.clinerules/documentation-lifecycle.md`

Use the same rule file from Solution 1, with this additional section:

```markdown
## Evidence pack priority

When a documentation evidence pack is provided:

- Treat the evidence pack as the bounded task scope.
- Do not expand beyond the changed files listed in the evidence pack unless required to identify existing documentation.
- If the evidence pack and repository contents disagree, report the inconsistency and ask the user how to proceed.
- Prefer exact diff excerpts and existing documentation excerpts over general repository knowledge.
```

### Python script: `tools/docs_lifecycle_pack.py`

```python
#!/usr/bin/env python3

import argparse
import json
import pathlib
import subprocess
from typing import List, Dict

SOURCE_EXTENSIONS = {
    ".py", ".ts", ".tsx", ".js", ".jsx", ".java", ".kt", ".cs", ".go",
    ".rs", ".rb", ".php", ".swift", ".scala", ".c", ".cc", ".cpp", ".h",
    ".hpp", ".sql", ".yaml", ".yml", ".toml"
}

DOC_EXTENSIONS = {".md", ".mdx", ".rst", ".adoc"}

def run(cmd: List[str]) -> str:
    return subprocess.check_output(cmd, text=True, stderr=subprocess.STDOUT)

def safe_run(cmd: List[str]) -> str:
    try:
        return run(cmd)
    except subprocess.CalledProcessError as exc:
        return exc.output

def changed_files(base_ref: str) -> List[str]:
    output = safe_run(["git", "diff", "--name-only", base_ref])
    return [line.strip() for line in output.splitlines() if line.strip()]

def git_diff(base_ref: str, file_path: str) -> str:
    return safe_run(["git", "diff", base_ref, "--", file_path])

def read_excerpt(path: pathlib.Path, max_chars: int = 12000) -> str:
    if not path.exists() or not path.is_file():
        return ""

    try:
        text = path.read_text(encoding="utf-8", errors="replace")
    except Exception as exc:
        return f"[Could not read file: {exc}]"

    if len(text) > max_chars:
        return text[:max_chars] + "\n\n[TRUNCATED]"
    return text

def find_doc_candidates(repo_root: pathlib.Path, terms: List[str], limit: int = 20) -> List[Dict]:
    docs = []
    for path in repo_root.rglob("*"):
        if ".git" in path.parts:
            continue
        if path.suffix.lower() not in DOC_EXTENSIONS:
            continue

        rel = str(path.relative_to(repo_root))
        text = read_excerpt(path, max_chars=20000)
        text_lower = text.lower()

        score = 0
        matched_terms = []
        for term in terms:
            if not term:
                continue
            normalized = term.lower()
            if normalized in text_lower or normalized in rel.lower():
                score += 1
                matched_terms.append(term)

        if score > 0 or rel.lower() in {"readme.md", "docs/index.md"}:
            docs.append({
                "path": rel,
                "score": score,
                "matched_terms": matched_terms[:20],
                "excerpt": text[:6000]
            })

    docs.sort(key=lambda item: item["score"], reverse=True)
    return docs[:limit]

def derive_terms(files: List[str]) -> List[str]:
    terms = set()
    for f in files:
        p = pathlib.Path(f)
        terms.add(p.stem)
        terms.add(p.name)
        for part in p.parts:
            if part not in {".", "src", "lib", "app", "test", "tests"}:
                terms.add(part)
    return sorted(t for t in terms if len(t) >= 3)

def main():
    parser = argparse.ArgumentParser()
    parser.add_argument("--base", default="HEAD", help="Base ref for git diff, e.g. HEAD, main...HEAD, origin/main...HEAD")
    parser.add_argument("--output", default=".docs_lifecycle_evidence.json")
    args = parser.parse_args()

    repo_root = pathlib.Path(run(["git", "rev-parse", "--show-toplevel"]).strip())
    files = changed_files(args.base)

    source_files = [
        f for f in files
        if pathlib.Path(f).suffix.lower() in SOURCE_EXTENSIONS
    ]

    doc_files = [
        f for f in files
        if pathlib.Path(f).suffix.lower() in DOC_EXTENSIONS
    ]

    terms = derive_terms(source_files)
    doc_candidates = find_doc_candidates(repo_root, terms)

    evidence = {
        "base_ref": args.base,
        "changed_files": files,
        "changed_source_files": source_files,
        "changed_documentation_files": doc_files,
        "source_diffs": [
            {
                "path": f,
                "diff": git_diff(args.base, f),
                "current_excerpt": read_excerpt(repo_root / f)
            }
            for f in source_files
        ],
        "documentation_candidates": doc_candidates,
        "instructions": {
            "scope": "Only documentation changes directly related to changed_source_files are allowed.",
            "accuracy": "Do not invent behavior. Use only source_diffs, current excerpts, and documentation candidates.",
            "approval": "Ask before applying documentation edits."
        }
    }

    output_path = repo_root / args.output
    output_path.write_text(json.dumps(evidence, indent=2), encoding="utf-8")
    print(str(output_path))

if __name__ == "__main__":
    main()
```

### Prompt 2: `.clinerules/workflows/docs-lifecycle-evidence-pack.md`

```markdown
# Documentation lifecycle from evidence pack

Use this workflow to update documentation from a structured Git evidence pack.

## Step 1: Generate evidence pack

Ask the user for the comparison base.

Default to:

```bash
python tools/docs_lifecycle_pack.py --base HEAD --output .docs_lifecycle_evidence.json
```

For pull-request style analysis, suggest:

```bash
python tools/docs_lifecycle_pack.py --base origin/main...HEAD --output .docs_lifecycle_evidence.json
```

## Step 2: Read the evidence pack

Open `.docs_lifecycle_evidence.json`.

Treat it as the authoritative task boundary.

Identify:

- Changed source files.
- Diffs.
- Current source excerpts.
- Candidate documentation files.
- Existing changed documentation files.

If there are no changed source files, stop.

## Step 3: Analyze documentation gaps

Produce this report:

```markdown
## Documentation gap analysis

### Evidence pack
- Base ref:
- Changed source files:
- Candidate docs:

### Findings
| Source file | Change visible in diff | Related doc | Gap type | Confidence |
|---|---|---|---|---|

### Required documentation actions
- Update:
- Create:
- No action:
- Needs human clarification:
```

Gap types must be one of:

- `missing`
- `outdated`
- `inconsistent`
- `none`
- `uncertain`

Use `uncertain` if the evidence is incomplete.

## Step 4: Draft changes

Draft documentation changes only for `missing`, `outdated`, or `inconsistent` gaps with medium or high confidence.

Follow the style of the target documentation.

Do not apply changes yet.

Ask the user to approve:

- Files to update.
- Files to create.
- Any uncertain items to skip.

## Step 5: Apply approved changes

After approval:

- Modify existing documentation where possible.
- Create new documentation only if no appropriate existing page exists.
- If creating a new page, keep it minimal and add it to an index only when an obvious index exists.

## Step 6: Verify

Run:

```bash
git diff --name-only
git diff --stat
git diff
```

Confirm that changes are limited to documentation files.

## Step 7: Clean up

Ask whether to keep or remove `.docs_lifecycle_evidence.json`.

If removing it, run:

```bash
rm .docs_lifecycle_evidence.json
```

## Step 8: Final response

Summarize:

- Source changes analyzed.
- Documentation gaps found.
- Documentation files