# Contributing to saas-security-end

Thanks for considering a contribution. This skill exists because real SaaS launches kept tripping over the same boring security failures — every PR that catches one more class of failure makes the next launch safer for someone else.

This guide covers what we want, what we don't, and how to ship a change.

---

## Code of Conduct

Be excellent to each other.

---

## Types of contributions most welcome

### Stack adapters
The current skill targets React + Supabase + Vercel. We want parallel versions (or branching prompts) for **Firebase**, **Convex**, **Pocketbase**, and **Appwrite**. An adapter rewrites the stack-specific checks (RLS becomes Firestore rules, Edge Functions become Cloud Functions, etc.) while keeping the 7-module structure intact.

### Compliance modules
The current scope is LGPD. We want sibling modules for **GDPR** (EU consent + data subject rights), **CCPA** (California-specific disclosure), and **HIPAA basics** (covered entity awareness, BAA reminders — not full HIPAA, that needs a specialist). Each new module should follow the same prompt-and-classify structure as Module 6.

### Gateway modules
Module 3 today is gateway-agnostic with examples from Asaas. We want specific check variants for **Stripe** (webhook signing with `stripe-signature`), **MercadoPago** (HMAC validation), **PagSeguro**, and **dLocal**. Each gateway has its own webhook quirks — encoding them once helps everyone.

### Example reports
Anonymized outputs from real audits are the best documentation. Strip project names, domains, table names, and any identifying detail, then drop the report into `examples/` so newcomers see what the skill actually catches.

### Bug fixes, typos, clarity improvements
Small PRs that fix a broken prompt, a confusing instruction, or a typo are always welcome — no issue needed.

### New modules
Categories we don't cover yet but would accept: **GraphQL security** (introspection, query depth, field-level auth), **mobile-specific** (deep linking, secure storage, cert pinning), **observability** (log hygiene, PII in traces, alert noise). Propose the module structure in an issue first.

---

## How to propose changes

1. **Open an issue first** for non-trivial changes (new modules, stack adapters, structural rewrites). This avoids you doing work that won't land.
2. **Fork** the repo, branch from `main`, and open a PR back to `main`. One logical change per PR.
3. **Keep prompts in the original language they were written in.** Most existing prompts are in PT-BR — leave them that way. New modules can be in EN or PT, whichever is more natural for the author.
4. **Test the skill locally before submitting.** Clone your fork into `~/.claude/skills/saas-security-end`, restart Claude Code, and run `/saas-security-end` against a sample project. If your change touches an existing module, verify the module still classifies correctly.

---

## Style guide for new modules

Every module should follow the same shape so the final report stays consistent.

- **Title** — short, numbered, and category-tagged (e.g. `MÓDULO 8 — OBSERVABILITY / Log Hygiene`).
- **Prompt block** — wrap the auditor prompt in triple backticks so Claude reads it verbatim. Write checks as numbered items with sub-bullets, not prose.
- **Expected result** — close every module with a `RESULTADO ESPERADO` (or `EXPECTED RESULT`) block listing what a passing audit looks like.
- **Classification system** — every check resolves to one of three states:
  - `✅ OK` — secure
  - `⚠️ Attention` — works but should improve
  - `❌ Vulnerable` — fix before launch
- **Generate fixes when possible.** If the issue is fixable in code, the prompt should instruct Claude to write the SQL, the diff, or the config change directly. If the fix is manual (DNS, dashboard toggle, legal review), list the exact steps.

---

## What NOT to add

- **Real production secrets or API keys**, even revoked ones. If you need to show an example token, use an obvious placeholder like `sk_live_REPLACE_ME`.
- **Project-specific names** in prompts or examples. Use generic placeholders (`my_table`, `mygateway.com`, `users`). The skill is meant to adapt to any project.
- **Backdoors, exploitation tooling, or offensive scripts.** This is a defensive skill. PRs that add attack payloads, scanners targeting third parties, or anything aimed at exploitation rather than defense will be rejected.

---

## Communication channel

For now: **GitHub Issues only.** No Discord, no Slack, no email list. Open an issue for discussion, questions, or proposals — that keeps the conversation searchable for the next contributor.
