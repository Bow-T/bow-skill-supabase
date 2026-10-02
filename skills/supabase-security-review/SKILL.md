---
name: supabase-security-review
description: Audit Supabase/backend changes (RLS, views, triggers, edge functions, SQL) for the recurring cross-tenant, secret and RLS mistakes, before committing. Use when changes touch supabase/ (migrations, policies, views, edge functions) or when the user asks to "check RLS", "security review the migration", "quest check", or "review supabase changes".
---

# Supabase Security Review

Catch the cross-tenant / secret / RLS mistakes before they reach review. Run the
automated gate first, then apply the manual checklist for what a grep-based gate
can't see. Authoring counterpart: [[supabase-migration]].

## 1. Run the automated gate
**If the repo has `scripts/check-quest.sh`:**
```bash
bash scripts/check-quest.sh --staged      # pre-commit: staged diff
bash scripts/check-quest.sh --branch      # whole MR: diff vs the base branch
```
Exit 0 = clean. On a finding it prints `✗ [rule] file` + the offending line.

**Otherwise** run `supabase db advisors` (or MCP `get_advisors`) and check the
staged diff by hand against the same four rules:
- **view-security-invoker** — `CREATE VIEW` on RLS tables must include `WITH (security_invoker = true)`, else it runs as owner and bypasses RLS.
- **cors-wildcard** — edge functions must not use `Access-Control-Allow-Origin: *` on admin/mutation endpoints.
- **secret-*** — no hardcoded Stripe live keys, Supabase service-role keys, OpenAI/Anthropic keys.
- **rls-with-check-user-id-only** — a `WITH CHECK` whose only predicate is `auth.uid() = user_id` (FK ownership of related rows not verified). Opt out only with an explicit comment when genuinely N/A (`-- du-quest: fk-not-applicable` in repos with the quest gate).

> Any finding is BLOCKING. Fix at the printed file:line and re-run.

## 2. Manual checklist (beyond the grep gate)
**RLS policies**
- INSERT/UPDATE `WITH CHECK` validates ownership of *every* related FK, not just `user_id`.
- Money / discount amounts are never trusted from the client — enforce server-side or via a `CHECK` constraint.
- Owner-read policies preserved (don't widen SELECT scope unintentionally).

**Triggers**
- `SECURITY DEFINER` triggers run with elevated rights → must pair with a strict INSERT policy on the table.
- Counter triggers are symmetric: increment on INSERT, decrement on DELETE/UPDATE; account for `ON DELETE CASCADE`.

**Edge functions**
- CORS scoped (no `*` for admin/mutation).
- Post-mutation side effects (logging, notifications) wrapped in `try/catch` so a secondary failure can't break the main DB transaction.

**SQL hygiene**
- No dead/meaningless clauses (e.g. `WHERE created_at < now()`).
- Backfills update the full related column-set (e.g. set `reviewed_at` whenever `reviewed_by` is set).

## 3. Permission helpers — verify the EFFECTIVE predicate
Two traps that static-green and a policy-text test never catch:

- **A broad permission key may not mean "the role holds that key".** A helper such as
  `has_permission(uid, 'broad.key')` can be written to pass when the caller holds
  ANY sub-key that the broad key expands to. Gating a policy on it then admits every
  role holding one of those sub-keys — a silent partial gate. To require a SPECIFIC
  role, use a direct role check (an `is_<role>(uuid)` helper pinned to that role),
  not a broad permission key.
- **Verify a helper against its LATEST definition, not the first grep hit.** Helpers
  get redefined across migrations. Read the newest body:
  ```bash
  grep -rln 'FUNCTION public.<fn>' supabase/migrations | sort | tail -1
  ```
  then confirm, for each role you mean to keep or block, that it passes or fails the
  predicate as it is defined THERE. Also compare with the edge function the policy
  is meant to mirror — the RLS gate must not be looser than it.

## 4. Report
For each item give `[✓/✗/N/A] <rule> <file:line> — note`. End with a verdict:
clean / list of blocking findings to fix before commit.
