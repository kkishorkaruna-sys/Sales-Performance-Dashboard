# Sales Performance Dashboard

A public, read-only sales dashboard backed by an administrator-published Excel or CSV workbook.

## Run & Operate

- `pnpm --filter @workspace/api-server run dev` — run the API server (port 5000)
- `pnpm run typecheck` — full typecheck across all packages
- `pnpm run build` — typecheck + build all packages
- `pnpm --filter @workspace/api-spec run codegen` — regenerate API hooks and Zod schemas from the OpenAPI spec
- `pnpm --filter @workspace/db run push` — push DB schema changes (dev only)
- Required environment variables: `DATABASE_URL`, `ADMIN_PASSWORD`, and `SESSION_SECRET` (keep values in Replit Secrets)

## Stack

- pnpm workspaces, Node.js 24, TypeScript 5.9
- API: Express 5
- DB: PostgreSQL + Drizzle ORM
- Validation: Zod (`zod/v4`), `drizzle-zod`
- API codegen: Orval (from OpenAPI spec)
- Build: esbuild (CJS bundle)

## Where things live

- `artifacts/sales-performance-dashboard` — public dashboard and protected workbook-publishing page
- `artifacts/api-server/src/routes/sales.ts` — admin session, workbook upload, and dashboard API
- `artifacts/api-server/src/lib/salesWorkbook.ts` — Excel/CSV parsing, header mapping, and formula-result diagnostics
- `lib/api-spec/openapi.yaml` — API contract; generated client and Zod types come from this file
- `lib/db/src/schema/salesDatasets.ts` — persisted active sales dataset
- `artifacts/sales-performance-dashboard/src/index.css` — dashboard theme and responsive layout

## Architecture decisions

- Uploaded workbooks are parsed in memory; only normalized sales rows and dataset metadata are stored in PostgreSQL.
- Formula cells use the workbook's saved results; the server does not execute spreadsheet formulas. Dashboard subtotals include only saved values, show missing-row counts, and never assume missing values are zero.
- The dashboard stays empty until an administrator validates and publishes a workbook; never substitute fabricated or sample sales values.
- Administrator sessions are signed, HTTP-only, same-site cookies; the expected password and session secret remain server-side.

## Product

- Visitors can filter sales by customer, product code, date range, hatchery, market, and area, and view KPIs and sales charts.
- Administrators can inspect workbook sheets, headers, and formula diagnostics, then publish one worksheet as the current dashboard dataset.
- OneDrive synchronization is not yet connected. Any future integration must read a private file server-side and must not make the workbook public.

## User preferences

- Keep the dashboard mobile-friendly and use a white, light-blue, and corporate-blue visual style.
- Keep the public dashboard read-only; workbook replacement is administrator-only.
- Review any Microsoft permissions, setup requirements, storage limits, and costs before implementing OneDrive synchronization.

## Gotchas

- `Total Orders` currently means data rows because the requested workbook fields do not include a distinct order identifier; disclose this if workbook structure changes.
- Prefer the `Total (Rs)` revenue field when present; do not add it to another total/amount column.
- Formula results can be stale or missing until the workbook is recalculated and saved in Excel.

## Pointers

- See the `pnpm-workspace` skill for workspace structure, TypeScript setup, and package details
