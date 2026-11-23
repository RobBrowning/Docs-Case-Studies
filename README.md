# Docs and Case Studies

Write-ups of how I test, with measured results. I'm a QA lead with 18+ years in testing, including
a Head of QA role, and I currently work on AI-assisted QA.

Each case study states what was tried, how it was measured (usually with seeded defects), the
numbers, and the limits of the evidence.

## Case studies

| Case study | Date | In one line |
|---|---|---|
| [AI in my QA workflow](case-studies/ai-in-my-qa-workflow/README.md) | Nov 2026 | Test agents, an AI PR reviewer and an independent result verifier, measured with seeded defects: 4/4 caught by the reviewer, 5 tester mistakes caught by the verifier, 0 false positives |
| [Automated evals and a QA plugin on the same AI chatbot](case-studies/evals-and-qa-plugin-on-an-ai-chatbot/README.md) | Nov 2026 | qa-toolkit vs a green eval suite on the same bot: 6 kinds of bug the suite missed on the clean build, 2 of 2 blind seeded defects named, and the verifier caught the tester in every run |
| [Testing an AI chatbot](case-studies/testing-an-ai-chatbot/README.md) | Nov 2026 | A RAG bot on local Hugging Face models, tested in layers: 7 real bot failures fixed one change per commit, a judge replaced after failing calibration, 5/5 seeded defects caught, hold-out 10/12 |
| [AI in QA: what I've built and learned, 2022 to 2026](case-studies/ai-in-qa-2022-2026/README.md) | Nov 2026 | The longer view: custom GPT suite, an AI manual tester plugin with an independent verifier, planning vs building with two Claudes, a small business's digital stack, and release dashboards |

## Playbooks

Reusable ways of working, written so another tester or team can adopt them. Project-specific names have been removed.

| Playbook | In one line |
|---|---|
| [AI-Assisted QA Playbook](playbooks/ai-assisted-qa/README.md) | The QA chain from ticket to sign-off with a blind verifier, read/write guardrails for AI on shared systems, test account rules and a new-project checklist |
| [Release Radar](playbooks/release-radar/README.md) | A read-only dashboard showing every open release's date, risk, environment and progress, rebuildable on any tracker and host |

## Related repos

- [playwright-qa-showcase](https://github.com/RobBrowning/playwright-qa-showcase): Playwright and TypeScript suite with test agents and AI PR review
- [cypress-qa-showcase](https://github.com/RobBrowning/cypress-qa-showcase): its Cypress twin
- [llm-qa-showcase](https://github.com/RobBrowning/llm-qa-showcase): testing an AI chatbot with layered evals, an independent judge and seeded defects

## Layout

```
case-studies/
  <case-study-name>/
    README.md      the write-up
    assets/        images or data, if any
playbooks/
  <playbook-name>/
    README.md      the playbook
```

New case studies go in their own folder under `case-studies/`, with a row added to the table above.
