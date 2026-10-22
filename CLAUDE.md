# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

A teaching exercise ("Clase 4 — DRY, KISS y YAGNI", OfiDocs internal billing/report utilities). The code is **deliberately smelly**; the task (see [README.md](README.md), written in Spanish) is to refactor it while keeping behavior. There is no app — just pure TypeScript modules in `src/`, tests in `test/`, and `src/demo.ts` to exercise them.

## Commands

```bash
npm install
npm test                          # vitest run (all tests)
npx vitest run test/pricing.test.ts          # single test file
npx vitest run -t "part of test name"        # single test by name
npm run typecheck                 # tsc --noEmit (covers src and test)
npm run demo                      # tsx src/demo.ts
```

No linter or build step. ESM project (`"type": "module"`, Node >= 22): imports use explicit `.ts` extensions (e.g. `from './types.ts'`).

## Architecture

Flat modules with no shared state; shapes live in `src/types.ts` (`Sale`, `Refund`, `Customer`, `Invoice`, `ShippingOrder`). `csv-exporters.ts` depends on `formatting.ts` (`formatDateSpanish`); `report-engine.ts` depends on `formatMoney`.

Refactoring targets and acceptance criteria (from the README):

- **DRY** — `validation.ts` has three near-identical email validators and `csv-exporters.ts` three near-identical exporters. The copies have *diverged*: some differences are real business rules (e.g. the newsletter validator rejects `+` aliases), others are copy-paste bugs (e.g. inconsistent trimming/null handling, leading-dot check missing in the contact validator; VAT rate `1.2` in `refundAmount` vs `1.21` elsewhere in `pricing.ts`). Classify each difference before unifying, and add a test for every rule you keep. Goal: one email validator, one CSV exporter, and the VAT rate defined exactly once in the project.
- **KISS** — `calculateShippingCost` (nested ifs; it is a rate table in disguise), and in `formatting.ts`: `formatDateSpanish`, `sortInvoicesByTotalDesc`, `invoiceLabel`, `formatMoney`. Goal: no function in `formatting.ts` over 10 lines; avoid "clever" regex replacements.
- **YAGNI** — `report-engine.ts` (~300 lines) exports caches, a plugin registry, an abstract renderer factory and retry policy, but the only real entry point is `generateReport(rows, title)`. Goal: delete unused code, keep behavior identical, file under 40 lines.

## Constraints

- Existing tests must **not be modified**; only add new ones (for rules/bugs discovered in the DRY step). Output format of `generateReport` and the CSVs is asserted exactly by tests.
- Per the README, document the list of differences found while unifying and the decision for each (in the PR description or a comment).
- Docs you write should be in English (user's global rule); the README is in Spanish.
