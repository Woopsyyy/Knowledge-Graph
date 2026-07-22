---
type: pattern-node
pattern: supabase-rls-rpc-hardening
domain: supabase / postgres / security
updated: 2026-06-23
tags: [pattern/supabase, rls, security-definer, grants, audit]
first-used: [[TCC-Portal]]
---

# Pattern: Supabase RLS / RPC hardening audit

For any app where the **browser talks to Supabase directly** (no backend server), the security
perimeter is **RLS policies + function `EXECUTE` grants**. Audit and harden against the **live DB**,
not the migration files (they drift). First applied on [[TCC-Portal]].

## Audit (read-only, via the Supabase MCP)

1. `get_advisors(type: security)` and `get_advisors(type: performance)` — the lint baseline.
2. Confirm every public table has RLS on:
   `select relname, relrowsecurity, (policy count) from pg_class … where relkind='r'`.
3. Inspect per-user scoping in `pg_policies` (qual / with_check) for sensitive tables — students must
   only match their own rows (`id/student_id/user_id = app_current_user_id()`).
4. **List SECURITY DEFINER functions and who can EXECUTE them** — this is where the real holes hide:
   ```sql
   select p.proname, p.prosecdef,
     array(select g.grantee::text from information_schema.role_routine_grants g
           where g.specific_schema='public' and g.routine_name=p.proname
             and g.grantee in ('anon','authenticated','public')) as grants
   from pg_proc p join pg_namespace n on n.oid=p.pronamespace
   where n.nspname='public' and p.prosecdef;
   ```
   Use `pg_get_functiondef(oid)` to read suspicious bodies.

## Red flags

- A SECURITY DEFINER function that **mutates auth/passwords or deletes data**, granted to `anon` or
  `authenticated`, **with no internal caller check** = privilege escalation / takeover. (Real example:
  `app_ensure_auth_user_for_password(user_id, password)` → set any account's password.)
- A login/lookup function that returns `row_to_json(users)` (leaks the password hash).
- Mutable `search_path` on SECURITY DEFINER fns (lint `0011`) — pin `set search_path = public`.
- `FOR ALL` "write" policies double up on SELECT (lint `0006`) — split to INSERT/UPDATE/DELETE.
- Unindexed foreign keys (lint `0001`).

## Fix recipe (idempotent migration)

- `drop function if exists` the leaky legacy fns.
- Revoke `anon` EXECUTE on all SECURITY DEFINER fns **except** a tiny pre-auth/public allowlist
  (login + public aggregate stats). Use a `DO` loop over `pg_proc … prosecdef` with an allowlist array.
- **Gotcha:** `revoke … from anon` does **not** remove an inherited `PUBLIC` grant — also
  `revoke … from public`. Verify with `has_function_privilege('anon', oid, 'EXECUTE')`.
- Fully revoke destructive/oracle helpers from anon **and** authenticated (they run as owner inside
  definer fns, so internal callers still work).
- Helpers used **inside RLS policies** (`app_current_user_id`, `app_has_role`, `app_is_*`) must keep
  the `authenticated` grant (the querying role needs EXECUTE to evaluate the policy).
- Re-verify with `get_advisors` after applying.

## Notes

- The `0028`/`0029` "anon/authenticated can execute SECURITY DEFINER" lints are expected for a
  no-backend app whose API *is* its RPCs — accept them once each fn enforces its own role/owner check.
- Publishable anon keys (`sb_publishable_…`) and PostHog `phc_…` ingest keys are safe to ship to the
  client; service-role and PostHog `phx_…` personal keys are not.
