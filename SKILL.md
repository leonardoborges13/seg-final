---
name: seg_final
description: Sprint de Seguranca Final para qualquer SaaS — auditoria completa de banco (RLS), autenticacao, pagamentos, frontend, infraestrutura, LGPD e testes manuais pre-lancamento. Stack: React + Supabase + Gateway de Pagamento + Vercel.
---

# /seg_final — Sprint de Seguranca Final (Pre-Lancamento)

Executa a auditoria de seguranca completa para preparar um SaaS para producao.

## Quando usar

- Antes de lancar qualquer SaaS em producao
- Quando quiser auditar seguranca de um projeto existente
- Quando adicionar pagamentos ou dados sensiveis a um projeto
- Sempre que mudar de sandbox para producao

## Como funciona

1. Leia o arquivo `Sprint_seg_final.md` neste mesmo diretorio
2. Identifique qual projeto esta sendo auditado (diretorio atual)
3. Execute os 7 modulos em ordem:
   - Modulo 1: Banco de Dados (RLS) — CRITICO
   - Modulo 2: Autenticacao — CRITICO
   - Modulo 3: Pagamentos — CRITICO
   - Modulo 4: Frontend — IMPORTANTE
   - Modulo 5: Infraestrutura — IMPORTANTE
   - Modulo 6: LGPD — IMPORTANTE
   - Modulo 7: Testes Manuais — gerar checklist personalizado

## Instrucoes de execucao

### Step 1: Ler o documento base
Leia `~/.claude/skills/seg_final/Sprint_seg_final.md` por completo.

### Step 2: Mapear o projeto
Identifique no projeto atual:
- Quais tabelas existem no Supabase (migrations, types, schema)
- Quais rotas existem (AppRoot, router, routes)
- Qual gateway de pagamento e usado (Asaas, Stripe, etc.)
- Quais Edge Functions existem
- Quais variaveis de ambiente sao usadas
- Qual plataforma de deploy (Vercel, Netlify, etc.)
- Quais dominios estao configurados

### Step 3: Executar cada modulo
Para cada modulo (1 a 6):
1. Adapte o prompt generico para as tabelas/rotas/funcoes reais do projeto
2. Execute a auditoria buscando no codigo real
3. Para cada item, classifique como:
   - ✅ OK — seguro
   - ⚠️ ATENÇÃO — funciona mas pode melhorar
   - ❌ VULNERÁVEL — precisa corrigir ANTES de lancar
4. Corrija automaticamente o que for possivel (codigo frontend)
5. Para correcoes no Supabase (RLS, policies), gere os SQLs prontos
6. Para correcoes na infra (Vercel, DNS), liste os passos manuais

### Step 4: Gerar checklist personalizado
No Modulo 7, gere o checklist de testes manuais PERSONALIZADO para o projeto:
- Use os dominios reais do projeto
- Use as rotas reais
- Use os nomes reais das tabelas e funcoes

### Step 5: Relatorio final
Gere um relatorio resumido:

```
RELATÓRIO DE SEGURANÇA — [Nome do Projeto]
Data: [data]

MÓDULO 1 — Banco: X/Y itens OK
MÓDULO 2 — Auth: X/Y itens OK
MÓDULO 3 — Pagamentos: X/Y itens OK
MÓDULO 4 — Frontend: X/Y itens OK
MÓDULO 5 — Infra: X/Y itens OK
MÓDULO 6 — LGPD: X/Y itens OK

VULNERABILIDADES ENCONTRADAS: N
CORRIGIDAS AUTOMATICAMENTE: M
PENDENTES (ação manual): K

[Lista de itens pendentes com instrucoes]
```

## Notas

- Esta sprint cobre seguranca para SaaS de baixo-medio risco
- Para compliance avancado (PCI-DSS, SOC2), consultar especialista
- A tokenizacao via gateway de pagamento cobre a maior parte do risco de pagamentos
- Sempre execute TODOS os 7 modulos — nao pule nenhum
- O Modulo 7 (testes manuais) deve ser feito pelo desenvolvedor no browser real
