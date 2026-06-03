# 🛡️ saas-security-end — SaaS Security Sprint (Pre-Launch)

> **A battle-tested Claude Code skill that audits the security of any SaaS before it ships to production.**
> Covers database (RLS), authentication, payments, frontend, infrastructure, LGPD compliance, and manual testing.

🇧🇷 [Versão em Português](./README.pt-BR.md)

---

## What it does

`/saas-security-end` runs a structured **7-module security audit** on your SaaS project, designed to catch the issues that actually kill product launches:

| Module | Focus | Priority |
|---|---|---|
| **1. Database** | Row-Level Security (RLS) on every table | 🔴 Critical |
| **2. Authentication** | Route protection, admin checks, session safety | 🔴 Critical |
| **3. Payments** | API keys, tokenization, webhook validation, server-side amounts | 🔴 Critical |
| **4. Frontend** | Env vars, XSS, dependencies, console leaks | 🟡 Important |
| **5. Infrastructure** | Security headers, CORS, HTTPS, backups | 🟡 Important |
| **6. LGPD / Data protection** | Consent, ToS/Privacy pages, right to deletion | 🟡 Important |
| **7. Manual tests** | Personalized pre-launch checklist | 🔴 Final gate |

For each item, Claude classifies as ✅ OK · ⚠️ Attention · ❌ Vulnerable — and **fixes what it can automatically** (frontend code, RLS SQL, env var moves).

---

## Stack assumed

This skill was built against the most common Brazilian SaaS stack, but the principles apply broadly:

- **Frontend:** React (Vite or Next.js)
- **Backend:** Supabase (Postgres + Auth + Edge Functions + Storage)
- **Payments:** Asaas, Stripe, or any tokenizing gateway
- **Deploy:** Vercel, Netlify, or similar
- **Compliance:** LGPD (Brazilian GDPR equivalent)

Works on any project that matches this shape. Easy to adapt to other stacks (Firebase, Pocketbase, etc.) — the prompts are mostly stack-agnostic.

---

## Installation

### Option 1 — Clone directly into your Claude skills folder (recommended)

```bash
# macOS / Linux
git clone https://github.com/leonardoborges13/saas-security-end.git ~/.claude/skills/saas-security-end

# Windows (PowerShell)
git clone https://github.com/leonardoborges13/saas-security-end.git $env:USERPROFILE\.claude\skills\saas-security-end
```

That's it. Restart Claude Code and type `/saas-security-end` to invoke it.

### Option 2 — Manual install

1. Download `SKILL.md` and `playbook.md` from this repo
2. Create folder `~/.claude/skills/saas-security-end/`
3. Drop both files inside
4. Restart Claude Code

### Option 3 — Per-project install

If you want the skill only in one project:

```bash
mkdir -p .claude/skills/saas-security-end
curl -O https://raw.githubusercontent.com/leonardoborges13/saas-security-end/main/SKILL.md
curl -O https://raw.githubusercontent.com/leonardoborges13/saas-security-end/main/playbook.md
mv SKILL.md playbook.md .claude/skills/saas-security-end/
```

---

## Usage

Inside any project directory in Claude Code:

```
/saas-security-end
```

Claude will:

1. **Read the playbook** (`playbook.md`)
2. **Map your project** — discovers tables, routes, Edge Functions, env vars, deploy platform, domains
3. **Run all 7 modules in order**, adapting each prompt to your real code
4. **Fix what it can automatically** and generate SQL/manual steps for the rest
5. **Generate a personalized manual test checklist** with your real routes and domain names
6. **Output a final security report** with health score and pending items

Typical run takes 15–40 minutes depending on project size.

---

## When to run it

- ✅ Before launching a new SaaS to production
- ✅ When auditing an existing project's security posture
- ✅ When adding payments or sensitive data
- ✅ When switching from sandbox to production gateway
- ✅ Quarterly, as a regression check

---

## What it does NOT cover

This skill targets **low-to-medium-risk SaaS** (under ~10k MAUs, no PHI, no PII at scale).

For advanced compliance, consult a specialist:
- **PCI-DSS** (handling raw card data — tokenization sidesteps this)
- **SOC 2** (organizational + process controls beyond code)
- **HIPAA** (healthcare data)
- **Penetration testing** (red team / formal pentest)

The skill explicitly flags this limit in its final report.

---

## Sample output

```
SECURITY REPORT — MyApp.com
Date: 2026-06-03

MODULE 1 — Database (RLS): 12/14 items OK
  ⚠️ Table `user_documents` has RLS enabled but no SELECT policy → blocks everything
  ❌ Table `payment_methods` has RLS DISABLED — CRITICAL, fix before launch
  → SQL fix generated in `migrations/sec-001-payment-methods-rls.sql`

MODULE 2 — Auth: 8/8 items OK ✅
MODULE 3 — Payments: 6/7 items OK
  ❌ Webhook endpoint not validating Asaas token → CRITICAL
  → Fix applied in `supabase/functions/payment-webhook/index.ts:42`

MODULE 4 — Frontend: 11/12 items OK
MODULE 5 — Infrastructure: 4/6 items OK
MODULE 6 — LGPD: 3/5 items OK
  ⚠️ No "delete my account" flow exists
  ⚠️ Privacy policy missing data retention section

VULNERABILITIES FOUND: 6
AUTO-FIXED: 3
PENDING (manual action): 3
SHIP READINESS: 🟡 BLOCKED on 2 critical items
```

---

## Philosophy

Most SaaS security failures aren't sophisticated attacks — they're:
- A forgotten table without RLS
- An API key accidentally committed
- A webhook that accepts any payload
- A `plan_status` field that the frontend can write to

This skill is a **systematic sweep** of the boring, high-impact failure modes. It won't find a zero-day. It will find the things that get exploited by anyone with browser DevTools and 5 minutes.

---

## Contributing

PRs welcome. The most useful contributions:

- **Stack adapters** — versions for Firebase, Convex, Pocketbase, Appwrite
- **Compliance modules** — additions for GDPR, CCPA, HIPAA basics
- **Gateway modules** — specific checks for Stripe, MercadoPago, PagSeguro, dLocal
- **Example reports** — anonymized reports from real audits showing what the skill catches

Open an issue first for big changes so we can align on scope.

---

## License

[MIT](./LICENSE) — use it, fork it, sell consulting with it. Attribution appreciated but not required.

---

## Credits

Built by [Leonardo Borges](https://github.com/leonardoborges13).

Forged from real pre-launch audits. Each module exists because something went wrong on a real project.

> *"Para que possais executar a vossa salvação com temor e tremor."* — Filipenses 2:12
