# Docs and Case Studies

Write-ups of how I test, with measured results. I'm a QA lead with 18+ years in testing, including
a Head of QA role, and I currently work on AI-assisted QA.

Each case study states what was tried, how it was measured (usually with seeded defects), the
numbers, and the limits of the evidence.

## Case studies

| Case study | Date | In one line |
|---|---|---|
| [AI in my QA workflow](case-studies/ai-in-my-qa-workflow/README.md) | Nov 2026 | Test agents, an AI PR reviewer and an independent result verifier, measured with seeded defects: 4/4 caught by the reviewer, 5 tester mistakes caught by the verifier, 0 false positives |
| [AI in QA: what I've built and learned, 2022 to 2026](case-studies/ai-in-qa-2022-2026/README.md) | Nov 2026 | The longer view: custom GPT suite, an AI manual tester plugin with an independent verifier, planning vs building with two Claudes, a small business's digital stack, and release dashboards |

## Related repos

- [playwright-qa-showcase](https://github.com/RobBrowning/playwright-qa-showcase): Playwright and TypeScript suite with test agents and AI PR review
- [cypress-qa-showcase](https://github.com/RobBrowning/cypress-qa-showcase): its Cypress twin

## Layout

```
case-studies/
  <case-study-name>/
    README.md      the write-up
    assets/        images or data, if any
```

New case studies go in their own folder under `case-studies/`, with a row added to the table above.
