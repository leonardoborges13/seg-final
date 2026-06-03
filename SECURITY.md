# Security Policy

## Reporting a Vulnerability

If you find a security-relevant issue in this skill, please report it privately to **nborges1313@gmail.com**. Do not open a public GitHub issue for security reports — we want a chance to assess and correct guidance before it is widely seen.

In the context of this project, a "vulnerability" means:

- A technical error in an audit prompt that could lead a user to **mis-configure security** (for example, a suggested SQL/RLS fix that opens a hole instead of closing it).
- Outdated guidance that contradicts current security best practices for the assumed stack.
- A missing check for a known, critical vulnerability class in the stack this skill targets (React + Supabase + payment gateway + Vercel).

The following are **not** security reports — please open a regular GitHub issue instead:

- Typos, wording, or clarity improvements → use GitHub Issues.
- Suggestions for new modules, checks, or sections → use GitHub Issues.
- Requests for adapters to additional stacks (e.g., Firebase, Auth0, Stripe variants) → use GitHub Issues.

**Expected response:** best-effort. This is a project maintained by one person. Aim for an acknowledgment within 7 days of your email.

## Supported Versions

| Version | Supported     |
|---------|---------------|
| 1.x     | ✅ Active      |
| < 1.0   | ❌ (none — 1.0 is the first public release) |

## Out of Scope

- This is a **skill** (a set of markdown prompts). There is no running server, no stored user data, and no authentication surface in this repository.
- The skill does **not** execute arbitrary code on user systems beyond what Claude Code itself does when running a skill.
- The skill does **not** make network requests on its own.
- Issues with Claude Code itself: report to Anthropic, not here.

## Defensive Use Only

This skill exists to **help people secure their applications**. It is not a checklist of attack vectors for systems you do not own or have explicit permission to test. The project follows the principle of responsible defensive tooling — please use it that way.
