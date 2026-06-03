# Sprint de Segurança Final — Pré-Lançamento SaaS

> **Objetivo**: Garantir que o sistema está seguro para receber usuários reais, processar pagamentos e proteger dados
> **Prioridade**: BLOQUEANTE — sem isso, não lança
> **Aplicável a**: Qualquer SaaS com React + Supabase + Gateway de Pagamento + Vercel/Deploy

---

## POR QUE ESTA SPRINT EXISTE

Antes de lançar um SaaS que:
- Coleta dados pessoais (nome, email, CPF)
- Processa pagamentos (cartão de crédito, Pix)
- Armazena informações financeiras de usuários
- Tem sistema de login e autenticação

...precisamos garantir que ninguém consegue: roubar dados, acessar contas de outros usuários, burlar o trial/pagamento, ou comprometer a integridade do sistema.

---

## MÓDULO 1 — AUDITORIA DE SEGURANÇA DO BANCO (Supabase)

### Prompt para executar:

```
AUDITORIA DE SEGURANÇA DO BANCO DE DADOS

Verificar TODAS as tabelas do projeto e garantir:

1. RLS (Row Level Security) — CRÍTICO:

   Para CADA tabela do projeto, verificar:
   a) RLS está HABILITADO? (ALTER TABLE ... ENABLE ROW LEVEL SECURITY)
   b) Existem policies criadas?
   c) As policies filtram por auth.uid() = user_id?
   d) Não existe nenhuma tabela com dados sensíveis sem RLS

   Listar TODAS as tabelas e o status de RLS de cada uma:
   
   SELECT tablename, rowsecurity 
   FROM pg_tables 
   WHERE schemaname = 'public';

   Se QUALQUER tabela com dados de usuário estiver sem RLS = VULNERABILIDADE CRÍTICA.
   
   Classificação de tabelas:
   - Tabelas de USUÁRIO: devem ter RLS com auth.uid() = user_id
     (products, sales, stock_movements, subscriptions, payment_history, 
     tax_settings, ai_usage, ai_credits, product_images, etc.)
   - Tabelas de DADOS PADRÃO: podem ter RLS mais flexível
     (marketplaces, marketplace_fees — dados default visíveis para todos)
   - Tabelas ADMIN: apenas admins leem/escrevem
     (admin_users, admin_action_logs, webhook_logs, system_config, etc.)

2. Policies de INSERT:
   Verificar se usuários só podem inserir dados com SEU PRÓPRIO user_id.
   Ninguém pode inserir dados no nome de outro usuário.
   
   Exemplo correto:
   CREATE POLICY "Users insert own data" ON products
     FOR INSERT WITH CHECK (auth.uid() = user_id);

3. Policies de UPDATE:
   Verificar se usuários só podem atualizar SEUS PRÓPRIOS dados.
   
4. Policies de DELETE:
   Verificar se usuários só podem deletar SEUS PRÓPRIOS dados.
   Considerar se soft-delete é melhor que hard-delete para algumas tabelas.

5. Verificar se a tabela auth.users NÃO está exposta via API pública.
   Dados sensíveis de auth nunca devem vazar para o frontend.

6. Verificar se campos sensíveis não estão expostos:
   - Tokens de cartão de crédito (NUNCA retornável via API)
   - IDs de clientes do gateway (desnecessário no frontend)
   
   Se necessário, criar view que exclui campos sensíveis.

7. Verificar se NÃO existe nenhuma função com SECURITY DEFINER 
   que possa ser explorada. Funções SECURITY DEFINER executam com 
   privilégios do criador (superuser), então devem ser mínimas e 
   muito bem validadas.

RESULTADO ESPERADO:
- Lista de todas as tabelas com status RLS ✅ ou ❌
- Lista de policies por tabela
- Correções aplicadas para qualquer falha encontrada
- Nenhum dado sensível exposto via API pública
```

---

## MÓDULO 2 — SEGURANÇA DE AUTENTICAÇÃO

### Prompt para executar:

```
AUDITORIA DE AUTENTICAÇÃO

1. PROTEÇÃO DE ROTAS:
   Verificar que TODAS as rotas do sistema (exceto landing page, 
   login e registro) estão protegidas por autenticação.
   
   Listar todas as rotas do projeto e classificar:
   - 🟢 Pública (não precisa login): /, /login, /register, /reset-password
   - 🔒 Protegida (precisa login): /dashboard, /app/*
   - 🔴 Admin (precisa login + is_admin): /admin, /admin/*
   
   Para cada rota protegida, verificar:
   a) Existe verificação de sessão/token antes de renderizar?
   b) Se não logado, redireciona para /login?
   c) Se logado mas sem acesso (ex: trial expirado), redireciona para paywall?

2. PROTEÇÃO DO ADMIN:
   Verificar que /admin e todas as sub-rotas:
   a) Verificam se user_id está na tabela admin_users
   b) Se não é admin, redireciona com "Acesso negado"
   c) Não basta ser "logado" — precisa ser ADMIN
   d) A verificação acontece no SERVIDOR (RLS/Edge Function), 
      não apenas no frontend (frontend pode ser burlado)

3. SESSÃO E TOKENS:
   a) Tokens do Supabase Auth expiram corretamente?
   b) Refresh token está sendo usado?
   c) Logout limpa todos os dados de sessão?
   d) Não há tokens armazenados em localStorage sem necessidade?

4. FORMULÁRIOS DE LOGIN/REGISTRO:
   a) Rate limiting: Não permite tentativas infinitas de login
   b) Senha mínima: 8 caracteres (verificar validação)
   c) Email de confirmação: está ativo?
   d) Reset de senha: funciona corretamente?
   e) Não expõe se email existe ou não na mensagem de erro
      (ex: "Email ou senha incorretos" ao invés de "Email não encontrado")

5. VERIFICAÇÃO DE TRIAL/PAGAMENTO:
   a) A verificação de acesso roda no SERVIDOR?
   b) Não é possível burlar o trial alterando dados no localStorage?
   c) A data do trial é validada no banco, não no frontend?
   d) Não é possível criar múltiplas contas para ter trial infinito?

RESULTADO ESPERADO:
- Mapa de todas as rotas com classificação de acesso
- Confirmação de que admin é verificado server-side
- Nenhuma vulnerabilidade de autenticação aberta
```

---

## MÓDULO 3 — SEGURANÇA DE PAGAMENTOS

### Prompt para executar:

```
AUDITORIA DE SEGURANÇA DE PAGAMENTOS

1. API KEYS:
   a) Verificar que API keys do gateway de pagamento estão APENAS nas 
      Edge Functions (Deno.env.get / process.env), NUNCA no código frontend
   b) Buscar em TODO o projeto por strings que pareçam API keys:
      Buscar: "ak_", "sk_", "pk_", "$aact_", "Bearer"
      Se encontrar no frontend = VULNERABILIDADE CRÍTICA
   c) Verificar que webhook tokens estão apenas na Edge Function do webhook

2. TOKENIZAÇÃO DE CARTÃO:
   a) Dados do cartão (número, CVV, validade) NUNCA passam pelo 
      seu servidor/Edge Function
   b) Tokenização é feita no frontend via SDK do gateway (client-side)
   c) Apenas o TOKEN encriptado é enviado para a Edge Function
   d) O token do cartão armazenado no banco não é acessível via API 
      pública (verificar RLS)
   e) Buscar em todo o projeto por: "creditCard", "cardNumber", 
      "card_number", "cvv", "ccv"
      Se esses dados são logados ou armazenados = VULNERABILIDADE CRÍTICA

3. WEBHOOK:
   a) A Edge Function do webhook VALIDA o token de autenticação 
      enviado pelo gateway no header?
   b) Verificar idempotência: o mesmo webhook processado 2x não 
      duplica pagamento (checar por payment_id único)
   c) O endpoint do webhook não aceita qualquer payload sem validação
   d) Erros no webhook são logados para debug

4. VALORES E FRAUDE:
   a) O valor da cobrança é definido no SERVIDOR (Edge Function), 
      não vem do frontend
      Se o frontend envia o valor = usuário pode alterar para R$0.01
   b) Descontos são calculados no servidor, não no frontend
   c) Parcelas são validadas no servidor
   d) Cupons são validados server-side (não confiar no frontend)

5. STATUS DE PAGAMENTO:
   a) O plan_status é atualizado APENAS pelo webhook (server-side),
      nunca pelo frontend
   b) Não é possível o frontend chamar um endpoint que mude 
      plan_status para 'active' sem pagamento real
   c) Verificar que não existe rota/função que ative acesso 
      sem confirmação de pagamento

RESULTADO ESPERADO:
- Confirmação de que API keys nunca vazam pro frontend
- Dados de cartão nunca tocam no servidor
- Valores de cobrança definidos server-side
- Webhook validado e idempotente
- Nenhum bypass de pagamento possível
```

---

## MÓDULO 4 — SEGURANÇA DO FRONTEND

### Prompt para executar:

```
AUDITORIA DE SEGURANÇA DO FRONTEND

1. VARIÁVEIS DE AMBIENTE:
   a) Listar TODAS as variáveis de ambiente usadas no projeto
   b) Verificar quais estão expostas no frontend (VITE_* / NEXT_PUBLIC_*) 
      e quais estão apenas no servidor
   c) Variáveis que PODEM estar no frontend (são públicas por natureza):
      - VITE_SUPABASE_URL / NEXT_PUBLIC_SUPABASE_URL
      - VITE_SUPABASE_ANON_KEY (é pública, RLS protege os dados)
   d) Variáveis que NUNCA devem estar no frontend:
      - API keys de gateways de pagamento
      - Webhook tokens
      - SUPABASE_SERVICE_ROLE_KEY
      - API keys de IA (OpenAI, Anthropic, etc.)
   e) Buscar em TODO o código por: process.env, import.meta.env
      Verificar se algo sensível está exposto

2. PROTEÇÃO CONTRA XSS:
   a) Verificar que inputs de texto do usuário são sanitizados
      antes de renderizar (React já faz escape por padrão com JSX,
      mas verificar se há uso de dangerouslySetInnerHTML)
   b) Buscar por: dangerouslySetInnerHTML
      Se existir: verificar se o conteúdo é sanitizado com DOMPurify
   c) Verificar que dados vindos da API são tratados antes de exibir

3. PROTEÇÃO CONTRA INJEÇÃO:
   a) Verificar que queries ao Supabase usam os métodos do SDK 
      (.select(), .eq(), .insert()) e NÃO SQL raw concatenado
   b) Buscar por: .rpc( — verificar que funções RPC validam inputs
   c) Buscar por: sql`, template literals com SQL = risco de injeção

4. DADOS SENSÍVEIS NO CONSOLE:
   a) Buscar por: console.log que contenha dados sensíveis:
      - Tokens, API keys, senhas
      - Dados de cartão
      - CPF, dados pessoais
   b) Em produção, console.logs com dados sensíveis devem ser removidos

5. DEPENDÊNCIAS:
   a) Rodar: npm audit
   b) Verificar se há vulnerabilidades HIGH ou CRITICAL
   c) Atualizar dependências com vulnerabilidades graves
   d) Listar bibliotecas desatualizadas com falhas conhecidas

6. CORS E HEADERS:
   a) Verificar que as Edge Functions retornam headers CORS corretos 
      (apenas o domínio do projeto permitido)
   b) Verificar se não há Access-Control-Allow-Origin: * em 
      endpoints sensíveis (pagamento, webhook)

RESULTADO ESPERADO:
- Nenhuma variável sensível exposta no frontend
- Nenhum uso de dangerouslySetInnerHTML sem sanitização
- Nenhum console.log com dados sensíveis
- npm audit sem vulnerabilidades HIGH/CRITICAL
- CORS configurado corretamente
```

---

## MÓDULO 5 — SEGURANÇA DA INFRAESTRUTURA

### Prompt para executar:

```
AUDITORIA DE INFRAESTRUTURA

1. PLATAFORMA DE DEPLOY (Vercel/Netlify/etc):
   a) Variáveis de ambiente estão corretas e separadas por environment?
   b) Nenhuma variável de servidor (API keys) na plataforma de deploy frontend
   c) Deploy automático está na branch correta?
   d) Preview deployments não acessam produção?
   e) HTTPS forçado?
   f) Headers de segurança configurados:
      - X-Content-Type-Options: nosniff
      - X-Frame-Options: DENY
      - X-XSS-Protection: 1; mode=block
      - Referrer-Policy: strict-origin-when-cross-origin
      - Permissions-Policy: camera=(), microphone=(), geolocation=()

2. SUPABASE:
   a) Service Role Key NÃO está em nenhum código frontend
   b) Anon Key está configurada com permissões mínimas
   c) RLS está ativo em todas as tabelas
   d) Backups automáticos estão habilitados?
   e) Logs de acesso estão habilitados?
   f) Rate limiting do Supabase Auth está configurado?

3. DOMÍNIO E DNS:
   a) Domínio aponta para plataforma de deploy corretamente?
   b) HTTPS é forçado (redirect de HTTP para HTTPS)?
   c) HSTS está configurado? (Strict-Transport-Security)
   d) Subdomínios desnecessários estão desativados?

4. EDGE FUNCTIONS / SERVERLESS:
   a) Todas as Edge Functions validam autenticação antes de processar?
   b) Edge Functions de pagamento têm rate limiting?
   c) Edge Functions retornam erros genéricos, não detalhes internos?
   d) Timeout está configurado adequadamente?

RESULTADO ESPERADO:
- Headers de segurança configurados
- Nenhuma chave sensível na plataforma de deploy frontend
- HTTPS forçado
- Edge Functions validam auth e têm rate limiting
- Backups ativos
```

---

## MÓDULO 6 — PROTEÇÃO DE DADOS E LGPD

### Prompt para executar:

```
AUDITORIA DE PROTEÇÃO DE DADOS (LGPD)

O sistema coleta e processa dados pessoais de brasileiros.
A LGPD (Lei Geral de Proteção de Dados) se aplica.

1. DADOS COLETADOS — mapear:
   a) Nome completo (registro)
   b) Email (registro e login)
   c) CPF/CNPJ (configuração fiscal e pagamento)
   d) Dados de cartão (tokenizados via gateway, não armazenamos)
   e) Dados financeiros do negócio
   f) Endereço IP (logs de acesso)
   
2. CONSENTIMENTO:
   a) Existe checkbox de "Li e aceito os Termos de Uso e 
      Política de Privacidade" no registro?
   b) Registrar data/hora do aceite no banco
   c) Links para Termos de Uso e Política de Privacidade 
      devem existir e ser acessíveis

3. PÁGINAS OBRIGATÓRIAS (criar se não existem):
   a) /termos — Termos de Uso
   b) /privacidade — Política de Privacidade
   
   Conteúdo mínimo da Política de Privacidade:
   - Quais dados coletamos
   - Para que usamos
   - Com quem compartilhamos (gateway de pagamento, etc.)
   - Como protegemos (criptografia, RLS, tokenização)
   - Direitos do usuário (acessar, corrigir, deletar dados)
   - Como solicitar exclusão de dados
   - Contato do responsável

4. DIREITO DE EXCLUSÃO:
   a) Existe funcionalidade para o usuário deletar sua conta?
   b) Ao excluir: anonimizar dados pessoais, cancelar assinatura,
      manter registros financeiros anonimizados (obrigação fiscal)

5. COOKIES:
   a) Se usa analytics: precisa de banner de cookies com consentimento
   b) Se usa apenas cookies essenciais de autenticação:
      banner não é obrigatório, mas menção na Política de Privacidade é necessária

RESULTADO ESPERADO:
- Mapa de dados pessoais coletados
- Checkbox de aceite no registro
- Páginas de Termos e Privacidade criadas
- Funcionalidade de exclusão de conta
- Conformidade básica com LGPD
```

---

## MÓDULO 7 — TESTES DE SEGURANÇA PRÁTICOS

### Executar manualmente ANTES do lançamento:

```
CHECKLIST DE TESTES MANUAIS DE SEGURANÇA

Executar cada teste e marcar como ✅ ou ❌:

1. AUTENTICAÇÃO:
   [ ] Acessar rota protegida sem login → deve redirecionar para /login
   [ ] Acessar /admin sem login → deve redirecionar para /login
   [ ] Acessar /admin logado como usuário normal → "Acesso negado"
   [ ] Fazer login com senha errada 10x → verificar se há rate limiting
   [ ] Fazer logout → verificar que não consegue voltar com botão "voltar"
   [ ] Criar conta com email já existente → deve dar erro

2. ISOLAMENTO DE DADOS:
   [ ] Logar como Usuário A, anotar ID de um registro
   [ ] Logar como Usuário B, tentar acessar registro do Usuário A 
       via URL direta ou chamada API → deve retornar vazio/erro
   [ ] Verificar que Usuário B não vê dados de Usuário A em nenhuma tela

3. TRIAL E PAGAMENTO:
   [ ] Criar conta → verificar que trial está ativo
   [ ] Verificar que validação de trial é server-side 
       (alterar data local NÃO deve funcionar)
   [ ] Abrir DevTools → Console → tentar alterar plan_status 
       via Supabase client → deve ser bloqueado pelo RLS
   [ ] Verificar que não existe endpoint que ativa conta sem pagamento

4. PAGAMENTO:
   [ ] Abrir DevTools → Network → verificar que dados de cartão 
       NÃO aparecem nas requisições para seu servidor
   [ ] Verificar que o valor da cobrança vem do servidor, 
       não do frontend (alterar valor no DevTools não deve funcionar)
   [ ] Simular webhook com payload falso (sem token válido) → 
       deve ser rejeitado

5. ADMIN:
   [ ] Acessar admin como admin → funciona
   [ ] Verificar que ações admin geram log de auditoria
   [ ] Tentar acessar endpoint admin via API sem ser admin → bloqueado

6. DADOS SENSÍVEIS:
   [ ] DevTools → Sources → buscar por "api_key", "secret", "token" 
       → nenhuma chave sensível no código
   [ ] DevTools → Network → nenhum token de API externo nos headers
   [ ] Verificar que console.log não imprime dados sensíveis

7. HTTPS:
   [ ] Acessar via http → deve redirecionar para https
   [ ] Verificar certificado SSL válido (cadeado verde)
```

---

## ORDEM DE EXECUÇÃO

| Módulo | Nome | Prioridade |
|--------|------|-----------|
| 1 | Banco de Dados (RLS) | 🔴 Primeiro |
| 2 | Autenticação | 🔴 Segundo |
| 3 | Pagamentos | 🔴 Terceiro |
| 4 | Frontend | 🟡 Quarto |
| 5 | Infraestrutura | 🟡 Quinto |
| 6 | LGPD | 🟡 Sexto |
| 7 | Testes Manuais | 🔴 Último (antes de lançar) |

---

## RESUMO DE PROTEÇÕES

| Ameaça | Proteção | Módulo |
|--------|----------|--------|
| Usuário acessa dados de outro | RLS no Supabase | 1 |
| Acesso sem login | Rotas protegidas | 2 |
| Usuário comum acessa admin | Verificação is_admin server-side | 2 |
| Roubo de API key | Keys apenas em Edge Functions | 3, 4 |
| Dados de cartão interceptados | Tokenização client-side | 3 |
| Alteração de valor de pagamento | Valor definido no servidor | 3 |
| Bypass de pagamento | Status atualizado apenas via webhook | 3 |
| Webhook falso | Validação de token do gateway | 3 |
| XSS (injeção de script) | React escape + sem dangerouslySetInnerHTML | 4 |
| SQL Injection | Supabase SDK (queries parametrizadas) | 4 |
| Dados sensíveis em logs | Limpeza de console.logs | 4 |
| Bibliotecas com falhas | npm audit | 4 |
| Ataque man-in-the-middle | HTTPS forçado + HSTS | 5 |
| Clickjacking | X-Frame-Options: DENY | 5 |
| Perda de dados | Backups automáticos | 5 |
| Descumprimento LGPD | Termos, Privacidade, exclusão de conta | 6 |
| Falha não detectada | Testes manuais pré-lançamento | 7 |

---

*Esta sprint cobre segurança para lançamento de um SaaS de baixo-médio risco com stack React + Supabase + Gateway de pagamento. Para compliance avançado (PCI-DSS, SOC2), consultar especialista em segurança.*
