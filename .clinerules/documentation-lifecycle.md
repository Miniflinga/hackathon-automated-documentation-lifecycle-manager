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
