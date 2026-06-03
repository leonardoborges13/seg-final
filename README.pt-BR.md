# 🛡️ seg-final — Sprint de Segurança SaaS (Pré-Launch)

> **Uma skill do Claude Code testada em produção que audita a segurança de qualquer SaaS antes de subir pra produção.**
> Cobre banco de dados (RLS), autenticação, pagamentos, frontend, infraestrutura, conformidade com LGPD e testes manuais.

🇺🇸 [English version](./README.md)

---

## O que faz

`/seg-final` executa uma **auditoria de segurança estruturada em 7 módulos** no seu projeto SaaS, desenhada pra pegar os problemas que realmente matam lançamento de produto:

| Módulo | Foco | Prioridade |
|---|---|---|
| **1. Banco de Dados** | Row-Level Security (RLS) em cada tabela | 🔴 Crítico |
| **2. Autenticação** | Proteção de rotas, validação de admin, segurança de sessão | 🔴 Crítico |
| **3. Pagamentos** | API keys, tokenização, validação de webhook, valores server-side | 🔴 Crítico |
| **4. Frontend** | Env vars, XSS, dependências, vazamentos no console | 🟡 Importante |
| **5. Infraestrutura** | Security headers, CORS, HTTPS, backups | 🟡 Importante |
| **6. LGPD / proteção de dados** | Consentimento, páginas de Termos/Privacidade, direito de exclusão | 🟡 Importante |
| **7. Testes manuais** | Checklist personalizado pré-launch | 🔴 Gate final |

Pra cada item, o Claude classifica como ✅ OK · ⚠️ Atenção · ❌ Vulnerável — e **corrige automaticamente o que dá** (código frontend, SQL de RLS, realocação de env vars).

---

## Stack assumida

A skill foi construída em cima da stack mais comum de SaaS brasileiro, mas os princípios se aplicam de forma ampla:

- **Frontend:** React (Vite ou Next.js)
- **Backend:** Supabase (Postgres + Auth + Edge Functions + Storage)
- **Pagamentos:** Asaas, Stripe, ou qualquer gateway com tokenização
- **Deploy:** Vercel, Netlify, ou similar
- **Compliance:** LGPD (equivalente brasileiro do GDPR)

Funciona em qualquer projeto que se encaixe nesse formato. Fácil de adaptar pra outras stacks (Firebase, Pocketbase, etc.) — os prompts são em sua maioria agnósticos à stack.

---

## Instalação

### Opção 1 — Clonar direto na sua pasta de skills do Claude (recomendado)

```bash
# macOS / Linux
git clone https://github.com/leonardoborges13/seg-final.git ~/.claude/skills/seg_final

# Windows (PowerShell)
git clone https://github.com/leonardoborges13/seg-final.git $env:USERPROFILE\.claude\skills\seg_final
```

É isso. Reinicia o Claude Code e digita `/seg_final` pra invocar.

### Opção 2 — Instalação manual

1. Baixe `SKILL.md` e `Sprint_seg_final.md` deste repo
2. Crie a pasta `~/.claude/skills/seg_final/`
3. Coloque os dois arquivos dentro
4. Reinicie o Claude Code

### Opção 3 — Instalação por projeto

Se quiser a skill em apenas um projeto:

```bash
mkdir -p .claude/skills/seg_final
curl -O https://raw.githubusercontent.com/leonardoborges13/seg-final/main/SKILL.md
curl -O https://raw.githubusercontent.com/leonardoborges13/seg-final/main/Sprint_seg_final.md
mv SKILL.md Sprint_seg_final.md .claude/skills/seg_final/
```

---

## Como usar

Dentro de qualquer diretório de projeto no Claude Code:

```
/seg_final
```

O Claude vai:

1. **Ler o playbook** (`Sprint_seg_final.md`)
2. **Mapear o projeto** — descobre tabelas, rotas, Edge Functions, env vars, plataforma de deploy, domínios
3. **Executar todos os 7 módulos em ordem**, adaptando cada prompt ao seu código real
4. **Corrigir automaticamente o que dá** e gerar SQL/passos manuais pro resto
5. **Gerar um checklist personalizado de testes manuais** com suas rotas reais e nomes de domínio
6. **Entregar um relatório final de segurança** com health score e itens pendentes

A execução típica leva de 15 a 40 minutos dependendo do tamanho do projeto.

---

## Quando rodar

- ✅ Antes de lançar um SaaS novo em produção
- ✅ Quando auditar a postura de segurança de um projeto existente
- ✅ Quando adicionar pagamentos ou dados sensíveis
- ✅ Quando migrar de sandbox pra gateway de produção
- ✅ Trimestralmente, como check de regressão

---

## O que NÃO cobre

Essa skill mira **SaaS de risco baixo a médio** (menos de ~10k MAUs, sem PHI, sem PII em escala).

Pra compliance avançado, consulte um especialista:
- **PCI-DSS** (manipulação de dados crus de cartão — tokenização contorna isso)
- **SOC 2** (controles organizacionais + de processo, além do código)
- **HIPAA** (dados de saúde)
- **Penetration testing** (red team / pentest formal)

A skill marca explicitamente esse limite no relatório final.

---

## Exemplo de output

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

## Filosofia

A maioria das falhas de segurança em SaaS não são ataques sofisticados — são:
- Uma tabela esquecida sem RLS
- Uma API key commitada por acidente
- Um webhook que aceita qualquer payload
- Um campo `plan_status` em que o frontend pode escrever

Essa skill é uma **varredura sistemática** dos modos de falha chatos e de alto impacto. Não vai achar um zero-day. Vai achar as coisas que são exploradas por qualquer um com o DevTools do navegador e 5 minutos.

---

## Contribuindo

PRs são bem-vindos. As contribuições mais úteis:

- **Adaptadores de stack** — versões pra Firebase, Convex, Pocketbase, Appwrite
- **Módulos de compliance** — adições pra GDPR, CCPA, básico de HIPAA
- **Módulos de gateway** — checks específicos pra Stripe, MercadoPago, PagSeguro, dLocal
- **Exemplos de relatórios** — relatórios anonimizados de auditorias reais mostrando o que a skill pega

Abra uma issue antes de mudanças grandes pra a gente alinhar escopo.

---

## Licença

[MIT](./LICENSE) — use, faça fork, venda consultoria com isso. Atribuição apreciada mas não obrigatória.

---

## Créditos

Construído por [Leonardo Borges](https://github.com/leonardoborges13) — engenheiro por trás de vários lançamentos de SaaS brasileiros (Calc360 Pro, Jornada do Escolhido, Denner Santos Visagismo, BrokerMKT Analytics, Bruno Marques Visagismo).

Forjado a partir de auditorias reais pré-launch. Cada módulo existe porque algo deu errado num projeto real.

> *"Para que possais executar a vossa salvação com temor e tremor."* — Filipenses 2:12
