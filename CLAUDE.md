# CLAUDE.md

## Project

Beginner-friendly JavaScript and Node.js learning documentation using the existing modified MkDocs template.

## Language

- Write all learning materials in Indonesian.
- Keep programming keywords, API names, and technical terminology in English.
- Explain unfamiliar technical terms when first introduced.
- Use simple language suitable for absolute beginners.

## Repository Rules

- Preserve the existing MkDocs architecture and design.
- Documentation source directory: `materi/`.
- Build output directory: `site/materi/`.
- Do not replace existing configuration without explicit approval.
- Inspect relevant files before modifying them.
- Avoid unnecessary dependencies.

## Content Standards

Every learning chapter must contain:

1. Learning objectives.
2. Introduction and motivation.
3. Concept explanation.
4. Syntax and structure.
5. Code examples.
6. Explanation of every important code component.
7. Expected output.
8. Step-by-step execution explanation.
9. Practical use cases.
10. Common mistakes.
11. Exercises.
12. Summary.

## Code Standards

- Use modern JavaScript syntax.
- Use Node.js for executable examples.
- Prefer ES Modules unless a chapter explicitly discusses CommonJS.
- Every code example must be executable when practical.
- Explain unfamiliar syntax before using it.
- Show expected terminal output.
- Do not introduce advanced concepts prematurely.
- Verify code examples before declaring them complete.

## Writing Standards

- One Markdown file per focused topic.
- Use `##` as the highest heading level.
- Use `###` for subsections.
- Use `####` for deeper subsections.
- Avoid unnecessary repetition.
- Use tables only when they improve comparison.
- Include practical examples rather than lengthy theoretical descriptions.

## Token Efficiency

- Work only on files explicitly related to the current task.
- Never read the entire repository unnecessarily.
- Never regenerate completed chapters.
- Reuse existing conventions and templates.
- Make minimal, focused changes.
- Do not produce lengthy progress reports.
- Report only changed files, validation results, and important issues.

## Workflow

1. Inspect relevant files.
2. Propose a short implementation plan.
3. Implement only the requested scope.
4. Validate examples and Markdown.
5. Report completion and unresolved issues.

## Validation

Use the existing repository build workflow.

Expected commands:

```bash
python -m mkdocs build
python scripts/copy-dashboard.py
```

Do not change the dashboard, theme, or deployment configuration unless requested.