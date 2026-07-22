---
type: project-node
project: TCC-Portal
stack: Vite + React 19 (TS) SPA / Supabase / Vercel
status: active
updated: 2026-06-23
tags: [project/tcc-portal, react, vite, supabase, rls, security, school-portal]
---

# TCC Portal

Talisay City College campus management portal. Repo in WSL at
`~/Project/School/Talisay-City-College-Portal` (Windows: `\\wsl.localhost\Ubuntu\...`).

## Architecture (the key mental model)

- **Client-only SPA.** Vite + React 19 + TypeScript + styled-components + Tailwind, React Router 7.
  Deployed to **Vercel as static files** with a catch-all SPA rewrite to `/index.html`
  (`vercel.json`). There is **no backend server in the repo.**
- The "backend" is **Supabase accessed directly from the browser** with the publishable anon key
  (`sb_publishable_…` in `.env`, `VITE_SUPABASE_*`), plus **4 Edge Functions** (service-role):
  `public-student-signup`, `set-auth-password`, `session-guard`, and the two `cached-*` stat
  functions. Single client module: `src/supabaseClient.ts`.
- **Consequence: Supabase RLS + RPC `EXECUTE` grants are the entire security perimeter.** Client-side
  checks are UX only. Supabase project ref **`tfxuzkumdjxpmmjkcjcp`** (region us). **Local and prod
  share the same DB** — every DB change is immediately live in production. (This project lives under a
  *different* Supabase account than [[shared-supabase]]; re-auth the Supabase MCP to reach it.)

## Auth model

- Custom username/password login via the `app_login(identifier, password)` RPC (SECURITY DEFINER,
  `anon`-callable). Passwords are bcrypt; a DB trigger mirrors `public.users` rows into Supabase
  `auth.users` so `auth.uid()` works and RLS can key off it (`app_current_user_id()` maps
  `auth.uid()` → `public.users.id`). 8-digit email OTP for browser verification + password reset.
- Roles: `admin`, `teacher`, `office`/staff (sub-roles: nt, osas, treasury, go, dean, faculty),
  `student`. Helper fns `app_is_admin/teacher/student/staff`, `app_has_role` drive RLS.
- Per-user RLS scoping is correct: students only see their own `users`/`grades`/`feedbacks`/
  `teacher_evaluations`/`study_load`/`user_assignments`/`notifications` rows; staff/teacher/admin
  have widened SELECT. Every public table has RLS enabled.

## 2026-06-23 security + perf + analytics hardening pass

Migrations live in the repo at `supabase/migrations/2026062312*` and were applied to live via the
Supabase MCP. See also `docs/SECURITY-FOLLOWUPS.md`.

- **CRITICAL fixed — account takeover.** `app_ensure_auth_user_for_password/_for_hash` were granted
  to `anon`, letting anyone set ANY account's auth password (incl. admin) then log in. Revoked.
- **Data leak fixed.** Dropped legacy `login_user` — it returned `row_to_json(users)` *including the
  password hash* to `anon`, had a mutable `search_path`, and referenced the dropped `email` column.
- Revoked `anon` EXECUTE on all SECURITY DEFINER RPCs except the intended allowlist
  (`app_login`, `app_repair_auth_profile_for_login`, `app_public_stats`, `app_public_landing_stats`);
  fully revoked the destructive `app_purge_expired_users()` and the password oracle
  `app_password_matches_user()`. Pinned `search_path` on the `app_is_*` helpers.
- **Key gotcha:** the **live DB had drifted from `supabase/supabase.txt`** (the 7286-line RLS patch).
  Audit against the live DB (MCP `get_advisors` + `pg_policies`/`pg_proc` queries), not the file.
  Also: `revoke … from anon` does NOT remove an inherited `PUBLIC` grant — revoke from `public` too.
- **Perf:** added 6 missing FK indexes; converted 16 `FOR ALL` write policies to explicit
  INSERT/UPDATE/DELETE (cleared all `multiple_permissive_policies` + `unindexed_foreign_keys` lints).
- **Cleanup:** deleted dead insecure `src/lib/supabase.ts` (had a `'your-anon-key'` fallback client).
- **Analytics:** wired **PostHog** (`posthog-js`, `src/services/analytics.ts`, US host) — captures
  `$pageview` (SPA, from `AppRoutes`), `login_succeeded/failed`, `otp_verified`, `signup`, `logout`,
  with `identify`/`reset` on auth. Key in `VITE_POSTHOG_KEY`. **PostHog MCP server pending the user's
  `phx_` personal key** (project id 322504, US Cloud).
- **Mobile:** login/landing verified overflow-clean at 390px (Playwright); student views already use
  responsive `minmax()` grids + breakpoints + `MobileSectionNavigator`. No fixes needed. A logged-in
  student pass is still pending (no test creds).

## Residual follow-ups (in `docs/SECURITY-FOLLOWUPS.md`)

- Enable Auth leaked-password protection (HaveIBeenPwned) in the dashboard.
- Set `EDGE_ALLOWED_ORIGINS` on the Edge Functions (Bearer-token auth, so low risk).
- `avatars` public bucket allows file LISTING — tighten after confirming admin storage-stats usage.
- `app_bind_auth_uid_for_user` lets an authenticated user claim a `users` row with NULL `auth_uid`;
  add an ownership check.

## Dev / build notes

- **Node lives in WSL via nvm** (`~/.nvm/versions/node/v24.17.0`), not on the non-interactive PATH.
  Run tooling as: `wsl bash -lc 'export PATH="$HOME/.nvm/versions/node/v24.17.0/bin:$PATH"; cd … && npm …'`.
  Do NOT run `npx`/`npm` from Windows over the `\\wsl.localhost\` UNC path — `cmd.exe` rejects UNC.
- `npm run dev` = `scripts/vite-dev-phone.mjs` (prints Local + Phone LAN URLs; binds 0.0.0.0). Windows
  Playwright reached it via the WSL eth0 IP (`172.28.114.213:<port>`), not `localhost` (forwarding flaky).
- `npm run build` = `vite build` (does NOT typecheck; `npm run typecheck` has pre-existing errors in
  test files + `ChatBot.tsx`, unrelated to this work).
