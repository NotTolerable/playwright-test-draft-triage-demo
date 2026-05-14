# AGENTS.md — Playwright Test Drift Triage Demo

## 1. Project Goal

Build a small, reviewable TypeScript proof-of-work demo for Playwright test drift triage. The demo analyzes a broken Playwright-style checkout test, a failure log, and an updated DOM snapshot, then suggests a safer selector repair with clear confidence scoring and human-reviewable reasoning.

This project is intended for outreach to Stably AI and should demonstrate understanding of AI-powered test maintenance concepts, Playwright-compatible tests, CI-friendly execution, and selector autofix workflows without recreating Stably's product or calling external AI services.

## 2. Demo Scope

The core scenario is a checkout flow test that previously used:

```ts
page.getByTestId("checkout-button")
```

The UI changed, so that test id is missing in the current DOM. The updated page still contains a checkout CTA, but with a different test id and visible text. The analyzer should:

- detect likely selector drift from the failure log;
- extract the broken selector from the Playwright-style test source;
- inspect the updated DOM snapshot;
- find likely replacement CTA candidates;
- prefer a safer role/name locator when supported by the DOM;
- score confidence in the recommendation;
- generate a suggested patch rather than applying it silently;
- explain the reasoning in plain language.

Keep the demo intentionally small. It should be easy to read, run, and review in a few minutes.

## 3. Tech Stack

Use only the following core stack unless the user explicitly asks otherwise:

- TypeScript
- Node.js
- Vitest
- Playwright-style locator/test fixture examples
- Optional local CLI report using Node.js standard output

Avoid unnecessary runtime dependencies. If a dependency is added, it must have a clear purpose and be covered by tests or demo output.

## 4. Folder Structure

Preferred structure for the implementation, once built:

```text
.
├── AGENTS.md
├── README.md
├── package.json
├── tsconfig.json
├── vitest.config.ts
├── fixtures/
│   ├── broken-checkout.spec.ts
│   ├── checkout-failure.log
│   └── updated-checkout-dom.html
├── src/
│   ├── analyzer.ts
│   ├── candidateScoring.ts
│   ├── domSnapshot.ts
│   ├── failureLog.ts
│   ├── patchSuggestion.ts
│   ├── report.ts
│   └── types.ts
├── tests/
│   ├── analyzer.test.ts
│   ├── candidateScoring.test.ts
│   ├── failureLog.test.ts
│   └── patchSuggestion.test.ts
└── scripts/
    └── demo.ts
```

Keep fixtures small and realistic. Do not add generated artifacts, large snapshots, or unrelated sample applications.

## 5. Coding Style Rules

- Use TypeScript with explicit exported types for analyzer inputs and outputs.
- Prefer small pure functions for parsing, scoring, and patch generation.
- Keep side effects at the CLI/reporting boundary.
- Use descriptive names such as `extractMissingTestId`, `findCheckoutCandidates`, and `suggestLocatorPatch`.
- Avoid broad abstractions that make the demo harder to understand.
- Do not put `try`/`catch` blocks around imports.
- Do not call external services from the analyzer.
- Do not add hidden mutation of test files; suggestions must be returned as data or printed as a report.
- Favor deterministic behavior so tests are stable in CI.

## 6. Testing Rules

- Use Vitest for unit tests.
- Prioritize tests for parsing, candidate discovery, scoring, and patch suggestion behavior.
- Include fixture-based tests for the core checkout drift scenario.
- Tests should assert both the suggested replacement and the explanation/reason codes when practical.
- Avoid browser automation for this demo unless explicitly requested; DOM snapshots are sufficient.
- Keep test names behavior-oriented, for example: `suggests role/name locator when checkout test id drifts`.
- Run the verification commands before finalizing implementation changes when a runnable project exists.

## 7. Analyzer Rules

The analyzer should be deterministic and should focus on the checkout selector drift scenario. It should:

- detect missing `getByTestId(...)` selectors from Playwright-style failure logs;
- extract the broken selector from the test source;
- compare the missing selector against the updated DOM snapshot;
- identify interactive candidates such as buttons and links;
- detect checkout intent using accessible role, visible text, aria-label, id, test id, and nearby attributes;
- prefer Playwright locators that reflect user-visible behavior over brittle implementation selectors;
- return structured output containing the broken selector, candidates, selected suggestion, confidence score, patch text, and reasoning.

Do not design this as a general-purpose natural language agent. The demo should be scoped, explainable, and testable.

## 8. Candidate Scoring Rules

Candidate scoring should be simple, transparent, and documented in code. Prefer additive scoring with reason codes. Suggested scoring factors:

- strong positive score for accessible role `button` or `link` when appropriate;
- strong positive score for visible text or accessible name containing checkout-related terms;
- positive score for attributes such as `data-testid`, `id`, `name`, or `aria-label` containing checkout-related terms;
- positive score when the element is interactive and visible in the snapshot;
- negative score for disabled, hidden, unrelated, or low-intent elements;
- negative score for candidates that only match implementation details with no user-visible checkout intent.

Confidence levels should be easy to interpret, for example:

- `high`: clear checkout CTA with role/name match;
- `medium`: likely checkout CTA but with weaker evidence;
- `low`: possible match but should be reviewed carefully;
- `none`: no safe suggestion.

Prefer `page.getByRole("button", { name: /checkout/i })` or another role/name locator when the DOM supports it. Do not prefer a changed test id unless there is no stronger user-facing locator.

## 9. Patch Suggestion Rules

Patch suggestions must be reviewable and non-destructive. The tool should:

- generate a minimal diff-like suggestion or replacement snippet;
- show the old locator and suggested new locator;
- include confidence and reasoning;
- never auto-apply changes silently;
- avoid changing unrelated test code;
- avoid fabricating selectors that are not supported by the DOM snapshot;
- include a warning when confidence is not high;
- keep output suitable for a human reviewer to copy into a test.

For the core scenario, a good suggestion should replace the missing test id locator with a role/name locator when the updated DOM exposes a checkout CTA by accessible name.

## 10. CLI/Reporting Rules

A CLI report is optional but useful. If implemented, it should:

- run locally with `npm run demo`;
- read the included fixtures;
- print a concise report showing the broken selector, evidence, ranked candidates, confidence, and suggested patch;
- exit successfully for the expected demo scenario;
- avoid network calls and avoid writing to external services;
- make clear that the patch is a suggestion requiring review.

Do not build a web dashboard unless explicitly requested.

## 11. Features Not to Build

Do not build any of the following for this demo:

- a full Stably clone;
- real Stably API integration;
- real GitHub, pull request, or issue integration;
- LLM API calls;
- automatic silent patch application;
- authentication;
- database storage;
- deployment infrastructure;
- queue workers;
- browser farm orchestration;
- broad multi-framework test repair;
- large synthetic applications unrelated to the checkout drift scenario.

## 12. Documentation References

Use these references for conceptual alignment and README links:

- Stably documentation: https://docs.stably.ai/
- Stably website: https://www.stably.ai/
- Playwright locators: https://playwright.dev/docs/locators
- Playwright assertions: https://playwright.dev/docs/test-assertions
- Vitest guide: https://vitest.dev/guide/

When implementation details depend on external documentation, prefer official documentation and cite it in README prose where appropriate.

## 13. Verification Commands

When the runnable demo exists, verify with:

```bash
npm install
npm run test
npm run demo
```

If the repository is not yet implemented, do not invent passing results. Clearly state which commands were not applicable and why.

## 14. README Expectations

The README should explain:

- the purpose of the Playwright test drift triage demo;
- how the checkout selector drift scenario works;
- what inputs are analyzed: broken test, failure log, and updated DOM snapshot;
- how candidate scoring works at a high level;
- why role/name locators are preferred over brittle test-id replacement when possible;
- how to run `npm install`, `npm run test`, and `npm run demo`;
- example CLI output or a short sample report;
- explicit limitations, including no real Stably API, no LLM API, no GitHub integration, and no silent autofix.

Keep the README concise, practical, and oriented toward reviewers evaluating the demo.
