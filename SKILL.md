# Lein-Labz — Skill Completa do Projeto

## Contexto Geral

**Projeto:** Lein-Labz - Squad de Inovação em Negócios Digitais
**Equipe:** 4 estudantes FIAP
**Duração:** 30 dias - Intensive Business Building
**Objetivo Principal:** Construir 3-4 produtos SaaS/agência de serviços simultâneos

O Lein-Labz é um intensivo de construção de negócios onde um pequeno squad de FIAP constrói múltiplos projetos de forma paralela, cada um com modelo de negócio diferente, tecnologia distinta, e go-to-market independente. A stack é moderna, cloud-native, com ênfase em automação IA, integração de APIs de terceiros, e monetização rápida.

### Três Focos de Negócio
1. **Agência de Automação IA** - Serviços para SMBs (revenue share do squad)
2. **WhatsApp SaaS MVP** - Produto SaaS multi-tenant (produto de venda próprio)
3. **CyberShield Security** - Auditoria LGPD/segurança como serviço
4. **Produtos Digitais** - Courses, templates, assets

### Valores Essenciais
- **Shipping over Perfection:** MVP em dias, não semanas
- **Customer-First:** Sales e feedback dirigem desenvolvimento
- **Automação Total:** Workflows devem ser n8n-first, não código custom
- **Data Privacy:** LGPD compliance desde o dia 1, não é "depois"
- **Documentação Clara:** Cada decisão tem motivo técnico e comercial anotado

---

## Fase 1: Setup e Infraestrutura (Dias 1-3)
### O que Construir

**Objetivos:**
- Infra escalável pronta para 3 produtos
- Autenticação centralizada com Supabase
- CI/CD automático (GitHub → Vercel)
- Stripe + PIX configurado
- Monorepo com apps Next.js independentes
- Notion como produto database centralizado

**Entregas:**
- 3 repos Next.js no Vercel (cada produto em sua branch)
- Supabase project com auth, RLS policies e edge functions
- Stripe account br com PIX ativado
- n8n self-hosted ou cloud com 3+ workflows-base
- GitHub org com CI/CD setup

### Pesquisa: Supabase RLS + Auth para Multi-Tenant

**Fonte:** [Supabase Docs - Row Level Security](https://supabase.com/docs/guides/database/postgres/row-level-security)

#### Row Level Security (RLS) - Fundamentos

RLS é o coração da segurança em arquitetura multi-tenant. Cada tabela deve ter RLS habilitado por padrão. Supabase ativa RLS automaticamente em todas as tabelas criadas via dashboard.

**Regra de Ouro:** Use `auth.uid()` e `auth.jwt()` nas policies, NUNCA confie em dados do `user_metadata` porque end-users conseguem modificar isso em tempo real.

#### Performance com RLS

Performance é crítica. Índices fazem diferença exponencial:

```sql
-- Sempre crie índice na coluna user_id usada nas policies
CREATE INDEX idx_user_id ON your_table USING btree (user_id);
-- Melhoria vista: até 100x em tabelas grandes
```

**Padrão de Otimização - Evite JOINs em Policies:**
Em vez de fazer JOIN entre tabelas source e target na policy, organize a lógica para usar IN/ANY operations:

```sql
-- NÃO Lento: JOIN em policy
CREATE POLICY select_policy ON projects
  FOR SELECT TO authenticated
  USING (
    EXISTS (
      SELECT 1 FROM team_members
      WHERE team_members.team_id = projects.team_id
      AND team_members.user_id = auth.uid()
    )
  );

-- ✅ Rápido: Fetch data em array, use ANY
CREATE POLICY select_policy ON projects
  FOR SELECT TO authenticated
  USING (
    team_id = ANY(
      (SELECT array_agg(team_id) FROM team_members WHERE user_id = auth.uid())
    )
  );
```
#### RLS + Multi-Tenant Architecture

Para SaaS com múltiplos tenants:

1. **Adicionar coluna `tenant_id` em TODAS as tabelas públicas:**
```sql
ALTER TABLE projects ADD COLUMN tenant_id UUID NOT NULL;
ALTER TABLE users ADD COLUMN tenant_id UUID NOT NULL;
```

2. **Armazenar tenant no JWT custom claim:**
```javascript
// No signup/login, adicione ao user_metadata
await supabase.auth.updateUser({
  data: { tenant_id: 'org-uuid' }
})
```

3. **Policy padrão (copiar em todas as tabelas):**
```sql
CREATE POLICY tenant_isolation ON your_table
  FOR ALL TO authenticated
  USING (tenant_id = (auth.jwt() -> 'user_metadata' -> 'tenant_id')::uuid);
```

#### Boas Práticas Supabase Auth

- **Service Keys** (com `role: service_role`) NUNCA vão no browser. Apenas em backend seguro para admin operations.
- **Anon Keys** para anonymous users (read-only geralmente).
- **Usar `TO authenticated` explicitamente** nas policies para bloquear anon users automaticamente.
- **Refresh tokens:** Armazene em HTTPOnly cookies, não localStorage.

---

### Pesquisa: Vercel + Next.js Deploy Best Practices

**Fonte:** [Vercel Docs - Environment Variables](https://vercel.com/docs/environment-variables), [Next.js Docs](https://nextjs.org/docs/pages/guides/environment-variables)

#### Environment Variables - Regra Principal

**Client-side:** Prefix `NEXT_PUBLIC_` (visível ao navegador)
**Server-side:** Sem prefix (seguro, apenas servidor)
**API keys:** NUNCA exponha ao cliente.

```env
# .env.local (não commit)
DATABASE_URL=postgresql://...
STRIPE_SECRET_KEY=sk_live_...

# Pode committar (públicos)
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_live_...
NEXT_PUBLIC_API_URL=https://api.example.com
```

#### Vercel Environment Setup

Vercel oferece 3 tipos de ambiente:
- **Production:** Branch main (users reais)
- **Preview:** PRs automaticamente (teste antes de merge)
- **Development:** Local com `vercel env pull .env.local`

**Obrigatório:** Após adicionar nova env var, faça redeploy. Não sobe automaticamente.

#### Sistema de Variáveis Recomendado

```
VERCEL_ENV=production|preview|development (automático)
VERCEL_GIT_COMMIT_SHA=sha (automático)

# Stripe
NEXT_PUBLIC_STRIPE_PUBLISHABLE_KEY=pk_...
STRIPE_SECRET_KEY=sk_... (apenas em Vercel e local)

# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJ... (public)
SUPABASE_SERVICE_ROLE_KEY=eyJ... (admin only)

# APIs
NEXT_PUBLIC_API_URL=https://api.leinlabz.com
EVOLUTION_API_TOKEN=xxx (secreto)
N8N_WEBHOOK_URL=https://n8n.example.com/webhook/

# Analytics
NEXT_PUBLIC_POSTHOG_KEY=phc_...
NEXT_PUBLIC_SENTRY_DSN=https://...
```

#### Preview Deployments

Cada PR automaticamente gera URL única. Perfeito para testes antes de merge:
```
PR #42: https://pr-42-leinlabz.vercel.app
```

#### Limite de Variáveis

Máximo 64KB de env vars por deployment. Não é limite prático para SaaS pequeno.
---

### Pesquisa: Stripe Brasil - PIX + Assinaturas

**Fonte:** [Stripe - PIX Payments](https://docs.stripe.com/payments/pix), [Stripe Brasil Pricing](https://stripe.com/en-br/pricing)

#### PIX como Payment Method

PIX é o payment method mais usado no Brasil (40%+ das transações online). Stripe integra PIX nativamente desde 2024.

**Limitações de Transação:**
- Mínimo: 0.50 BRL
- Máximo: USD 3,000 por transação
- Limite monthly por cliente: USD 10,000 com qualquer merchant

#### Pix Automático para Subscriptions

Novidade em 2025: **Pix Automático** permite cobranças recorrentes (com consentimento do cliente) direto em PIX para subscriptions. Antes você precisava de cartão ou manual check-in.

**Implicação:** Você pode oferecer Lein-Labz SaaS com cobrança mensal em PIX puro.

#### Setup Técnico no Stripe

```javascript
// Habilitar PIX como payment method
const customer = await stripe.customers.create({
  email: 'user@example.com',
  payment_method_types: ['pix', 'card'], // PIX + Cartão
});

// Create subscription com PIX
const subscription = await stripe.subscriptions.create({
  customer: customer.id,
  items: [{ price: 'price_xxx' }],
  payment_settings: {
    payment_method_types: ['pix', 'card'],
  },
});
```

#### Pricing Information

Stripe Brasil oferece preços especiais para métodos de pagamento locais (PIX, boleto, etc), mas preços exatos não são públicos. Contato direto com Stripe sales necessário, mas historicamente PIX é mais barato que cartão.

**Estimado (não confirmado):** ~2-3% para PIX vs 2.99% + R$ 0.30 para cartão.

---

### Checklist de Setup - Dia 1-3

- [ ] **GitHub Org Setup**
  - [ ] Criar org `lein-labz`
  - [ ] 3 repos: `app-whatsapp`, `app-security`, `app-dashboard`
  - [ ] Branch protection: main requer PR + approval
  - [ ] GitHub Actions workflow base para build/test

- [ ] **Supabase Project**
  - [ ] Criar projeto `lein-labz-prod`
  - [ ] Tabelas base: `users`, `organizations`, `subscriptions`
  - [ ] RLS habilitado em todas as tabelas
  - [ ] Auth com email + OAuth (Google, GitHub)
  - [ ] Edge functions: `/functions/auth-webhook.ts` para sync user

- [ ] **Vercel Deployment**
  - [ ] Conectar 3 repos ao Vercel
  - [ ] Environment vars para production/preview/development
  - [ ] Custom domain: leinlabz.com (main), whatsapp.leinlabz.com, security.leinlabz.com
  - [ ] Analytics habilitado (Sentry + PostHog)

- [ ] **Stripe Account**
  - [ ] Account Stripe Brasil criado
  - [ ] PIX ativado como payment method
  - [ ] API keys configuradas em Supabase edge functions
  - [ ] Webhook configurado: `/api/webhooks/stripe`
  - [ ] Webhook topics: `payment_intent.succeeded`, `customer.subscription.updated`

- [ ] **n8n Setup**
  - [ ] Self-hosted em Render ou cloud.n8n.io
  - [ ] 3 workflows-base criados (vazios, prontos para automação)
  - [ ] Evolution API conectada
  - [ ] Stripe conectada
  - [ ] Webhook base: `/webhook/n8n-trigger` configurado

- [ ] **Monorepo Setup**
  - [ ] `pnpm workspaces` configurado
  - [ ] Shared components em `packages/ui`
  - [ ] Shared utils em `packages/lib`
  - [ ] Eslint + Prettier centralizado

---

## Fase 2: Agência de Automação IA (Dias 1-30, contínuo)

### Modelo de Negócio

**Proposta de Valor:** Lein-Labz oferece automações IA customizadas para SMBs brasileiros que ainda usam processos manuais. Foco em WhatsApp, CRM, e-mail, integrações de dados.

**Cliente Ideal:**
- Ecommerce (1-50 pessoas)
- Agências digitais
- Consultórios / Professional services
- Pequenos varejo online
- Revenda de serviços

**Problemas Que Resolvemos:**
1. Atendimento 24/7 que custa muito (usamos WhatsApp + Claude)
2. Lead qualification manual (automação + scoring)
3. Falta de integração entre sistemas legados
4. Marketing manual repetitivo

---

### Pesquisa: Mercado de Automação IA no Brasil

**Fonte:** [17 Top AI Automation Agencies 2025](https://latenode.com/blog/industry-use-cases-solutions/enterprise-automation/17-top-ai-automation-agencies-in-2025-complete-service-comparison-pricing-guide), [Zaytrics - AI Agency Business Model](https://zaytrics.com/ai-automation-agency-business-model-services-pricing-and-value-proposition/)

#### Tamanho do Mercado

Brasil tem ~303k WhatsApp Business customers (33% do global). A maioria são SMBs sem automação. Mercado de automação IA em Brasil ainda é nicho mas crescendo 300%+ ao ano.

#### Principais Oportunidades

1. **Integração WhatsApp + n8n + Claude:** Empresas querem chatbot inteligente mas não conseguem fazer.
2. **Lead Gen Automation:** Capturar leads de Facebook/Instagram, qualificar, enviar para WhatsApp.
3. **Data Sync Workflows:** Conectar Shopify → Supabase → WhatsApp → Stripe (pipeline completo).
4. **Document Processing:** PDFs, invoices, usando Claude vision.

#### Competidores Locais (Brasil)

- **Zenvia:** Plataforma matrixmm de CPaaS (cara, enterprise)
- **Take Blip:** Chatbot builder nocode (bom, mas caro: $1500+/mês)
- **Kommo:** CRM com messaging (força: já é CRM, fraco: automação limitada)
- **Yalo:** Conversational commerce (só varejo online)

**Oportunidade:** Nenhum competidor local oferece "automação IA + integração custom" por preço acessível. Lein-Labz pode dominar este nicho oferecendo projeto custom a $5k-15k com retorno em 2-3 meses.

---

### Pesquisa: Pricing de Serviços de Automação

**Fonte:** [AI Agency Pricing Guide 2025](https://digitalagencynetwork.com/ai-agency-pricing/), [Flexxable - How to Charge](https://flexxable.com/how-to-charge-clients-for-ai-services-pricing-models-for-ai-automation-agencies/)

#### Modelo #1: Project-Based (Inicial)

Cobrança fixa por projeto. Ideal para portfolio inicial.

```
Lead Gen Automation (WhatsApp + Scoring):
  Escopo: Setup Evolution API + n8n workflow + CRM sync
  Investimento Lein-Labz: ~60 horas
  Preço: R$ 20,000 - 30,000 (USD $4,000 - 6,000)
  Margem: ~60% (custo mensal infra ~5%)

Document Processing (PDFs com Claude Vision):
  Escopo: Setup, training, deployment
  Preço: R$ 15,000 - 25,000 (USD $3,000 - 5,000)

E-mail Campaign Automation:
  Preço: R$ 10,000 - 15,000 (USD $2,000 - 3,000)
```

#### Modelo #2: Retainer (Escalável)

Cobrança mensal para suporte + otimização. Melhor margem.

```
Base Retainer: R$ 2,000 - 4,000/mês
  - Monitoramento de workflow
  - 1 otimização/mês
  - Suporte técnico 48h

Premium Retainer: R$ 8,000 - 15,000/mês
  - Novos workflows (até 2/mês)
  - Otimização semanal
  - Suporte 24h

Projeção: 10 retainers × R$ 5,000 = R$ 50,000/mês = R$ 600,000/ano
```

#### Modelo #3: Revenue Share (Performance)

Lein-Labz cobra % da economia/receita gerada.

```
Exemplo: Ecommerce economiza R$ 50,000/ano com automação
  Lein-Labz pega 20% = R$ 10,000 (primeiro ano)
  + R$ 2,000/mês retainer = R$ 34,000/ano total
```

**Recomendação Squad:** Começar com Project-Based (prove value), escalar para Retainer (recurring revenue), depois oferecer Revenue Share para clientes maiores.

---

### Playbook de Vendas

#### Dia 1-5: Prospecção (Outreach)

**Alvo:** 30 SMBs no LinkedIn
- E-commerce (Shopify)
- Agências digitais
- Consultórios

**Template Primeira Mensagem:**
```
"Oi [Nome], vi que [empresa] usa WhatsApp para atender clientes.
A maioria das empresas ainda faz isso manualmente.

A gente automatiza atendimento via WhatsApp + IA (reduz custo de atendimento em 40-60%).
Posso mostrar um case em 15 min?

Abraços,
[Squad Member]"
```

#### Dia 5-10: Discovery Call

**Script Discovery:**
1. "Qual o maior pain point do seu atendimento hoje?"
2. "Quanto tempo sua equipe gasta nisso por semana?"
3. "Se economizasse 20h/semana, qual seria o impacto?"
4. "Vocês usam alguma ferramenta agora?" (CRM, chatbot)

**Objetivo:** Identificar oportunidade real. Se não há pain point claro, passe.

#### Dia 10-15: Proposta + Demo

**Proposta Estrutura:**
```
1. Diagnóstico (1 página)
   "Vocês recebem ~200 mensagens/dia, 4 pessoas resolvem
    Tempo médio: 3h/pessoa/dia = 12h/dia

2. Solução Lein-Labz (1 página)
   Bot WhatsApp + Claude (qualifica 80% dos leads)
   + Integração Supabase para CRM tracking
   + Escalonamento automático para 4 pessoas (casos complexos)

3. Resultados Esperados
   - 70% das mensagens resolvidas por bot
   - Redução de 8h/dia de atendimento
   - Custo: R$ 25,000 (setup) + R$ 3,000/mês (infra + suporte)
   - ROI: 3-4 meses

4. Timeline
   - Semana 1: Setup + treinamento
   - Semana 2: Beta com 20% de mensagens
   - Semana 3: 100% live + otimização
```

**Demo:** Mostrar prototype em 5 min. Usar exemplos reais da empresa deles se possível.

#### Dia 15+: Negotiation + Close

- Pedir 50% upfront, 50% na entrega
- Nunca faça grátis. Trabalho grátis = não valioso.
- Se não conseguir vender projeto, não continue. Mais clientes esperam.

---

### Stack Técnico para Projetos de Clientes

#### Core Stack por Tipo de Projeto

**Lead Gen + WhatsApp Automation:**
```
Evolution API → n8n → Claude API → Supabase → Stripe
[WhatsApp]    [Logic] [AI]   [Data] [Billing]
```

**Integrações Comuns:**
- **Entrada:** WhatsApp, Facebook Ads, Google Forms, Webhooks
- **Processamento:** n8n (orquestração), Claude (AI), Zapier (fallback)
- **Armazenamento:** Supabase (PostgreSQL)
- **Saída:** Slack, Email, CRM, Sheets, API webhook do cliente

**Exemplo Workflow Concreto:**
```
WhatsApp Message → Evolution API webhook
  ↓
n8n (trigger: evolution-api-webhook)
  ↓
Extract text intent (Claude API)
  ↓
Decision: Is lead? → If yes:
  ✓ Save to Supabase.leads
  ✓ Send to CRM via API
  ✓ Send score via WhatsApp (Claude response)
  ✓ Notify sales (Slack)
  ✓ Track in Stripe event (para futuro billing)
```

#### Stack Recomendado por Cliente Size

**Micro (1-10 pessoas):** n8n + Evolution API + Supabase
- Custo mensal: ~R$ 200 (Evolution grátis, n8n ~R$ 50, Supabase ~R$ 150)
- Setup: 2-3 dias

**Small (10-50 pessoas):** Acima + Slack + CRM integração + Claude API
- Custo mensal: ~R$ 1,500-2,000
- Setup: 1 semana

**Medium (50+):** Dedicated Supabase instance + n8n self-hosted + Edge functions custom
- Custo mensal: R$ 5,000+
- Setup: 2-3 semanas

---

### Pesquisa: n8n Workflows para Agências

**Fonte:** [n8n Community Workflows - 8515 templates](https://n8n.io/workflows/), [GitHub - awesome-n8n-templates](https://github.com/enescingoz/awesome-n8n-templates), [Contabo Blog - 10 n8n Best Practices](https://contabo.com/blog/10-n8n-best-practices-for-reliable-workflow-automation/)

#### Padrões Essenciais para Agência

**Padrão #1: Webhook Trigger → Process → Store → Notify**

```json
{
  "name": "Lead Capture - WhatsApp to CRM",
  "nodes": [
    {
      "type": "webhook",
      "name": "Evolution API Webhook",
      "event": "message.received"
    },
    {
      "type": "function",
      "name": "Extract Lead Info",
      "code": "return { name: data.contact.name, phone: data.contact.number }"
    },
    {
      "type": "postgres",
      "name": "Save to Supabase",
      "query": "INSERT INTO leads (name, phone, source) VALUES (...)"
    },
    {
      "type": "slack",
      "name": "Notify Sales",
      "message": "New lead: {{name}} - {{phone}}"
    }
  ]
}
```

#### Padrões de Erro + Retry

**NUNCA coloque workflow em produção sem tratamento de erro:**

```
Cada node crítico (API call, database) deve ter:
  ✓ Error handler local (retry 3x, wait 5s between)
  ✓ Fallback node (notify via email se falha)
  ✓ Logging (store error em Supabase para debug)
```

#### 5 Workflows Que Toda Agência Precisa

1. **Lead Qualification (WhatsApp → Score)**
   - Trigger: mensagem WhatsApp
   - Claude analisa intent (pergunta sobre serviço? Reclamação? Lead real?)
   - Envia score via WhatsApp
   - Salva em Supabase

2. **Email Campaign + Tracking**
   - Trigger: novo lead em Supabase
   - Enviar sequence email (day 1, 3, 7)
   - Track opens + clicks (Stripe webhook)
   - Auto-escalate se não abrir após 14 dias

3. **Invoice Processing (Automação Contábil)**
   - Trigger: Email com anexo PDF
   - Claude Vision lê invoice
   - Extrai: amount, date, vendor
   - Salva em Supabase, notifica contador via Slack

4. **Data Sync (Shopify → WhatsApp)**
   - Trigger: Nova venda em Shopify
   - Pull customer + order info
   - Enviar WhatsApp: "Pedido confirmado! Status: [...]"
   - Track delivery status, auto-update status

5. **Slack Notification Hub**
   - Centralize todas as notificações de clientes
   - Um único Slack conecta a N workflows
   - Template: `[Cliente] [Tipo] [Ação] [Link]`

#### Performance Best Practices

- **Modularidade:** Não faça workflow monolítico. Quebra em sub-workflows reutilizáveis.
- **Error Handling:** Retry com backoff exponencial (5s, 10s, 30s)
- **Logging:** Sempre salve input/output em Supabase para debug
- **Staging:** Teste workflow em preview antes de ligar em produção
- **Rate Limits:** Respeite limites de APIs (Evolution: 1000 msgs/min, Claude: 10 req/min)

---

## Fase 3: WhatsApp SaaS MVP (Dias 4-21)

### Produto

**Nome:** Lein-Labz Hub
**Tagline:** "Atendimento WhatsApp Inteligente para Pequenas Empresas"

**Proposta de Valor:**
- Bot WhatsApp com IA (Claude) integrado
- Escalação automática para humano quando necessário
- Dashboard simples: mensagens, leads, métricas
- Integração CRM básica (Supabase nativa)
- Suporte 24/7 via WhatsApp mesmo (meta-service)

**Diferencial:**
- Preço 70% mais barato que Take Blip (R$ 299/mês vs R$ 1500+)
- Feito por brasileiros, para brasileiros
- Claude + Automação (not just templates)
- Multi-language support: PT-BR, ES, EN

---

### Pesquisa: Evolution API (versão atual, setup, limites)

**Fonte:** [Evolution API v2 Documentation](https://doc.evolution-api.com/v2/en/integrations/cloudapi), [GitHub - EvolutionAPI](https://github.com/EvolutionAPI/evolution-api), [Gurusup - Evolution API 2026](https://gurusup.com/blog/evolution-api-whatsapp)

#### O Que é Evolution API

Evolution API é open-source, auto-hospedado, alternativa grátis à WhatsApp Official API. Oferece:
- Recebimento de mensagens
- Envio de mensagens
- Gerenciamento de mídia (imagens, documentos)
- Webhooks para eventos
- Multi-instance (múltiplos números)

**Importante:** Evolution não é official Meta. É mais barato, mais flexível, mas com trade-offs:
- Usa WhatsApp Web (menos estável que official API)
- Não é suportado por Meta oficialmente
- Risco: conta pode ser banida

**Recomendação:** Para produção com clientes reais, use official WhatsApp Business API. Evolution é perfeito para MVPs e protótipos.

#### Versão Atual (2026)

Evolution API v2 é versão stable com:
- REST API completa
- Webhook events
- Integration com WhatsApp Cloud API (oficial)
- Docker support
- Multi-instance management

#### Setup - 3 Cenários

**Cenário 1: Local Development (Rápido)**
```bash
docker run -d \
  -e PORT=3000 \
  -e EVOLUTION_INSTANCE_PATH=/evolution/instances \
  -p 3000:3000 \
  ghcr.io/evolutionapi/evolution-api:latest
```

Acesso: http://localhost:3000

**Cenário 2: Self-Hosted em Render/Railway (Recomendado MVP)**
```
- Deploy Evolution API docker em Render
- Connected WhatsApp number via QR code
- Webhook: https://yourdomain.com/api/webhooks/whatsapp
```

**Cenário 3: Official WhatsApp Business API (Produção)**
```
- Meta Business Manager → WhatsApp App
- Gerar access token (Admin user)
- Configurar webhook em Meta
- Usar Evolution API mode: "WHATSAPP-BUSINESS"
```

#### Limites Evolution API

| Métrica | Limite |
|---------|--------|
| Mensagens por minuto | 1,000 |
| Conexões simultâneas | 10,000+ (depende do servidor) |
| Tamanho máximo de mídia | 100MB |
| Tipos de mensagem | text, image, video, audio, document, button, list |

#### Webhook Events Importantes

```
- message.received: Nova mensagem chegou
- message.send: Mensagem enviada com sucesso
- message.update: Status da mensagem (delivered, read)
- connection.update: Conectado/desconectado
- chat.update: Chat foi criado/deletado
- contact.update: Contato adicionado/modificado
```

#### Integração com n8n

Evolution API + n8n = automação poderosa:

```json
{
  "webhook_url": "https://yourdomain.n8n.cloud/webhook/evolution",
  "events": ["message.received"],
  "node_type": "Webhook"
}
```

n8n node "Evolution API" disponível na community.

---

### Pesquisa: Mercado de WhatsApp SaaS no Brasil

**Fonte:** [Chakra Chat - Top 5 WhatsApp Solutions Brazil 2025](https://chakrahq.com/article/brazil-whatsapp-api-best-cheap-chakra-chat/), [Prelude - Top 10 WhatsApp BSPs](https://prelude.so/blog/best-whatsapp-business-solution-providers)

#### Market Size & Trends

Brasil: 303,331 WhatsApp Business customers (33.63% do global)
Growth: +300% YoY em automação
**Preço médio pago:** $30-150/mês

#### Concorrentes Diretos + Preço

| Competitor | Preço Entrada | Target | Força | Fraqueza |
|-----------|---------------|--------|-------|---------|
| Take Blip | R$ 1,500+/mês | Enterprise | Full automation | Caro, complex |
| Zenvia | R$ 2,000+/mês | Enterprise | CPaaS completo | Overkill para SMB |
| WATI | $30-40/mês | SMB | Simples, cheap | Limited AI |
| Chakra Chat | $12.49/mês | Ultra-budget | Super cheap | Básico |
| Kommo | $75/mês | SMB | CRM integrado | Marketing-first |
| Yalo | Custom | E-commerce | Conversational | Vertical-specific |

#### Oportunidade para Lein-Labz

**Sweet spot:** R$ 299-599/mês
- Mais barato que Take Blip (menos features complexas)
- Mais caro que WATI (mais AI, melhor UX)
- Foco: SMB e pequenas agências
- Diferencial: Claude integration + automação custom

**Mercado Addressable:**
- 303k WhatsApp Business customers × 10% penetration (market share potential)
- = 30,000 clientes × R$ 400/mês = R$ 12M/ano revenue potencial

---

### Pesquisa: Arquitetura Multi-Tenant com Supabase

**Fonte:** [Supabase RLS Docs](https://supabase.com/docs/guides/database/postgres/row-level-security), [Supabase Multi-Tenant Guide](https://supabase.com/docs/guides/realtime/examples/multi-tenancy)

#### Model: Organization-Based Multi-Tenancy

```sql
-- Core tables
CREATE TABLE organizations (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  name TEXT NOT NULL,
  owner_id UUID REFERENCES auth.users(id),
  stripe_customer_id TEXT UNIQUE,
  plan TEXT DEFAULT 'free', -- free, starter, pro
  created_at TIMESTAMP DEFAULT now()
);

CREATE TABLE organization_members (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  user_id UUID REFERENCES auth.users(id) ON DELETE CASCADE,
  role TEXT DEFAULT 'member', -- admin, member, viewer
  created_at TIMESTAMP DEFAULT now(),
  UNIQUE(organization_id, user_id)
);

CREATE TABLE whatsapp_instances (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  phone_number TEXT NOT NULL,
  evolution_instance_id TEXT NOT NULL,
  webhook_token TEXT,
  is_active BOOLEAN DEFAULT true,
  created_at TIMESTAMP DEFAULT now()
);

CREATE TABLE chats (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  whatsapp_instance_id UUID REFERENCES whatsapp_instances(id) ON DELETE CASCADE,
  contact_phone TEXT NOT NULL,
  contact_name TEXT,
  last_message_at TIMESTAMP,
  created_at TIMESTAMP DEFAULT now()
);

CREATE TABLE messages (
  id UUID PRIMARY KEY DEFAULT gen_random_uuid(),
  organization_id UUID REFERENCES organizations(id) ON DELETE CASCADE,
  chat_id UUID REFERENCES chats(id) ON DELETE CASCADE,
  sender_type TEXT, -- 'contact', 'system', 'agent'
  content TEXT NOT NULL,
  media_url TEXT,
  created_at TIMESTAMP DEFAULT now()
);
```

#### RLS Policies (Template Reutilizável)

```sql
-- RLS for all tables: Tenant isolation
CREATE POLICY org_isolation ON organizations
  FOR ALL TO authenticated
  USING (id IN (
    SELECT organization_id FROM organization_members
    WHERE user_id = auth.uid()
  ));

CREATE POLICY org_isolation ON organization_members
  FOR ALL TO authenticated
  USING (organization_id IN (
    SELECT organization_id FROM organization_members
    WHERE user_id = auth.uid()
  ));

CREATE POLICY org_isolation ON whatsapp_instances
  FOR ALL TO authenticated
  USING (organization_id IN (
    SELECT organization_id FROM organization_members
    WHERE user_id = auth.uid()
  ));

-- Similar para chats, messages
```

#### Auth Flow

```javascript
// 1. User signs up
const { data, error } = await supabase.auth.signUp({
  email: 'user@example.com',
  password: 'password123'
});

// 2. Create organization (in Next.js API route, use service role)
const { data: org } = await supabaseAdmin
  .from('organizations')
  .insert({ name: 'Minha Empresa', owner_id: userId });

// 3. Add user to organization
await supabaseAdmin
  .from('organization_members')
  .insert({ organization_id: org.id, user_id: userId, role: 'admin' });

// 4. On every request, fetch user's organizations (RLS handles filtering)
const { data: userOrgs } = await supabase
  .from('organizations')
  .select('*');
```

---

### Features do MVP (Dia 4-21)

#### Week 1: Core Infrastructure
- [ ] Deploy Evolution API (Render)
- [ ] Auth flow (sign up → create org → add WhatsApp number)
- [ ] Webhook receiver (Evolution → n8n → Database)
- [ ] Database schema (see above)
- [ ] Basic dashboard (React + TailwindCSS)

#### Week 2: Message Management
- [ ] Display incoming messages in dashboard
- [ ] Send message via dashboard (Evolution API)
- [ ] Chat history per contact
- [ ] Contact list + search
- [ ] Basic metrics (message count, active chats)

#### Week 3: AI + Automation
- [ ] Claude integration for bot responses
- [ ] Manual escalation (human takes over)
- [ ] Webhook from n8n → display bot response in UI
- [ ] Training data (customer provides examples for bot)
- [ ] Sentiment analysis (basic: positive/negative/neutral)

#### Week 4: Polish + Go Live
- [ ] Stripe integration (payment webhook)
- [ ] User invitation system (share organization)
- [ ] Analytics: messages/day, response time, bot accuracy
- [ ] Email notifications
- [ ] Security audit (LGPD checklist)
- [ ] Launch announcement + pre-sales

#### Scope Guardails (What NOT to build MVP)
- ❌ White-label support (later)
- ❌ Advanced analytics (dashboards)
- ❌ API for customer integrations (Phase 2)
- ❌ Multi-language bot (English only MVP)
- ❌ Advanced routing (simple escalation only)

---

### Go-to-Market (Dias 21-30)

#### Launch Strategy

**Week 1 (Dias 21-24): Soft Launch**
- Closed beta: 5-10 friendly customers (from Agência projects)
- Collect feedback: usability, bugs, feature requests
- Iterate based on feedback

**Week 2 (Dias 25-28): Public Launch**
- Product Hunt launch (prep: screenshots, demo video, copy)
- LinkedIn + Twitter announcement (squad members)
- Email outreach to prospects from sales pipeline
- 1-month free trial (get market data)

**Week 3 (Dias 29-30): Monetize**
- Pre-sales campaign: "Early bird: 50% off lifetime"
- Target: 50 signups month 1
- Monthly recurring: 50 × R$ 299 = R$ 14,950 MRR

#### Positioning

**Headline:** "ChatGPT para WhatsApp. Chatbot inteligente em 5 minutos."

**Copy:**
```
Seu atendimento WhatsApp cansa?
Lein-Labz autoatiza respostas com IA.

Seus clientes não veem a diferença.
Seu time economiza 20+ horas/semana.

Comece grátis. Sem cartão.
```

#### Metricas de Sucesso

- Month 1: 50 signups, 30% conversion para pago
- Month 2: 100 signups, 40% conversion
- Month 3: 150 signups, 50% conversion (proof: produto vale pena)

---

## Fase 4: CyberShield Security (Dias 7-30)

### Modelo de Negócio

**Serviço:** Auditoria de Segurança + LGPD Compliance para SaaS/Tech startups em Brasil

**Cliente Ideal:**
- Startups tech (Series A/B)
- SaaS brasileiras
- Agências com dados de clientes
- E-commerce com PCI compliance

**Problemas:**
- "Como ficamos LGPD compliant?"
- "Preciso de security audit para série A"
- "Qual ferramentas usar para pentesting?"
- "Quanto custa auditoria?"

**Solução:**
- Checklist LGPD customizado
- Penetration testing (ferramentas open source)
- Report profissional + recomendações
- Suporte pós-audit (implementação)

---

### Pesquisa: LGPD – Requisitos Essenciais

**Fonte:** [Planalto - Lei 13709](https://www.planalto.gov.br/ccivil_03/_ato2015-2018/2018/lei/l13709.htm), [Iugu Blog - LGPD para SaaS](https://www.iugu.com/blog/lgpd-para-saas), [Serpro - Como Cumprir LGPD](https://www.serpro.gov.br/lgpd/empresa/como-cumprir-a-lgpd)

#### Pilares Principais da LGPD

Lei 13.709 de 14 de agosto de 2018, em vigor desde setembro de 2020.

**Princípios (7 core):**
1. **Propósito e Adequação:** Coletar dados só para fins específicos, legitimados
2. **Livre Acesso:** Users conseguem ver quais dados você tem deles
3. **Qualidade de Dados:** Manter dados acurados, atualizados, completos
4. **Transparência:** Explicar claramente COMO você usa dados
5. **Prevenção:** Tomar medidas segurança PROATIVAS, não reactivas
6. **Não-Discriminação:** Não usar dados para discriminar
7. **Accountability:** Documentar TUDO. Ser capaz de provar compliance

#### Agentes Principais (3)

- **Controlador:** Pessoa/empresa que DECIDE como usar dados (você, normalmente)
- **Operador:** Pessoa/empresa que EXECUTA o uso de dados (Stripe, Supabase, etc.)
- **Data Protection Officer (DPO):** Pessoa responsável por compliance + auditor externo

#### 8 Bases Legais (Quando Você PODE Coletar)

```
1. Consent (Consentimento): User concorda explicitamente
2. Contract (Contrato): Necessário para entregar serviço
3. Legal obligation (Obrigação legal): Lei exige (ex: imposto)
4. Vital interests (Interesse vital): Salvar vida/saúde
5. Public interest (Interesse público): Bem comum
6. Legitimate interests (Interesses legítimos): Seu negócio (ex: prevenção fraude)
7. Credit: Análise de crédito/risco
8. Data protection: Próprio LGPD compliance
```

**Na prática SaaS:**
- Contato: Consent (ask permission)
- Usage metrics: Contract (necessário operar)
- Billing: Contract + Legal obligation

#### Direitos do Titular (Data Subject)

User pode:
1. **Access:** Ver todos seus dados em você
2. **Rectification:** Corrigir dados inexatos
3. **Deletion:** "Direito ao esquecimento" (com exceções)
4. **Portability:** Receber dados em formato aberto
5. **Opposition:** Não concordar com processing
6. **Restriction:** Limitar como você usa dados

**Prazo de resposta:** 15 dias úteis

#### Penalties por Violação

| Tipo | Penalidade |
|------|-----------|
| Leve | Até R$ 50,000 por violação |
| Grave | Até 2% receita anual ou R$ 50M (o que for maior) |
| Muito grave | Até 4% receita anual ou R$ 100M (o que for maior) |

**Importante:** ANPD (Autoridade Nacional Proteção Dados) fiscaliza ativamente.

---

### Pesquisa: Ferramentas Open Source de Security Scan

**Análise:** Não encontrada pesquisa web específica, mas baseado em conhecimento:

#### Ferramentas Recomendadas (Free/Open Source)

**SAST (Source Code Analysis):**
```
1. SonarQube: Vulnerabilities em código, code smells
   - Open source version disponível
   - Scan: JavaScript, Python, Java, etc.
   
2. Semgrep: Rápido, rulesets customizáveis
   - npm install semgrep
   - semgrep --config=p/security-audit .

3. Grype: Dependency scanning (vulnerabilities em libs)
   - Download: https://anchore.com/grype
   - grype dir:. (scan tudo)
```

**DAST (Dynamic Testing):**
```
1. OWASP ZAP: Web app security testing
   - docker run -t owasp/zap2docker-stable zap-baseline.py

2. Nikto: Web server scanner
   - nikto -h https://yourdomain.com
```

**Infrastructure:**
```
1. Trivy: Container image scanning
   - trivy image your-docker-image

2. Checkov: Infrastructure-as-Code scanning
   - checkov -d ./terraform (scan Terraform files)
```

**LGPD-Specific:**
```
- OWASP Privacy Breach Checklist
- OWPAS Data Protection Cheat Sheet
- (Não existe tool automatizada para LGPD; é manual)
```

#### Pipeline Security Recomendada

```yaml
# GitHub Actions (CI/CD)
name: Security Scan

on: [push, pull_request]

jobs:
  security:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3

      # Code scanning
      - name: SonarQube
        uses: SonarSource/sonarqube-scan-action@master

      # Dependency scanning
      - name: Grype
        run: grype dir:.

      # Docker image scanning
      - name: Trivy
        uses: aquasecurity/trivy-action@master
        with:
          image-ref: 'ghcr.io/lein-labz/app:latest'
```

---

### Pipeline de Auditoria (Serviço CyberShield)

#### Fase 1: Discovery (2 dias)
- [ ] Documentar arquitetura (tech stack, data flow)
- [ ] Listar todos endpoints API públicos
- [ ] Coletar documentação existente (API docs, security policies)
- [ ] Entrevista: "Como vocês guardam senhas? Logs? Backups?"

#### Fase 2: Automated Scanning (3 dias)
- [ ] Executar SAST (código)
- [ ] Executar DAST (API)
- [ ] Dependency scan (libs)
- [ ] Docker image scan
- [ ] Gerar relatório preliminar

#### Fase 3: Manual Testing (5 dias)
- [ ] Authentication/Authorization bypass attempts
- [ ] SQL Injection, XSS, CSRF
- [ ] Data exposure (logs, error messages)
- [ ] Cryptography review (secrets, encryption)
- [ ] Rate limiting, DDoS mitigation

#### Fase 4: LGPD Compliance Review (3 dias)
- [ ] Checklist LGPD compliance
- [ ] Data inventory (onde você guarda dados)
- [ ] Consent flows (legítima base para coleta)
- [ ] User rights implementation (access, delete, portability)
- [ ] Data processing agreements (com vendors)
- [ ] Incident response plan

#### Fase 5: Report + Recommendations (2 dias)
- [ ] Executivas summary
- [ ] Vulnerabilidades (critical, high, medium, low)
- [ ] LGPD compliance gaps
- [ ] Roadmap de fixes (com timeline)
- [ ] Q&A session com dev team

---

### Template de Relatório

```markdown
# Security & LGPD Audit Report
## [Client Name] - March 2026

### Executive Summary
[1 página]
- Overall risk: Medium
- Critical vulnerabilities: 3
- LGPD compliance: 60%
- Estimated remediation effort: 4 weeks

### Methodology
- OWASP Top 10 framework
- LGPD compliance checklist
- Tools: SonarQube, OWASP ZAP, Trivy, manual testing

### Findings

#### Critical (Fix immediately)
1. **SQL Injection in /api/users endpoint**
   - Location: src/api/users.js line 42
   - Impact: Database compromise, data theft
   - Fix: Use parameterized queries
   - Effort: 2 hours

2. **Hardcoded API key in .env.example**
   - Location: .env.example
   - Impact: Public exposure if checked in
   - Fix: Use Stripe secret manager
   - Effort: 1 hour

3. **No HTTPS enforcement**
   - Impact: Data interception
   - Fix: Add HSTS header, redirect http → https
   - Effort: 30 mins

#### High

#### Medium

#### Low

### LGPD Compliance Assessment

| Requirement | Status | Gap | Fix |
|------------|--------|-----|-----|
| Data consent | Partial | No opt-in form | Add modal |
| User access | No | Users can't download data | Build API endpoint |
| User deletion | No | No delete endpoint | 1 week |
| Data processing agreement | No | Missing vendor contracts | Stripe, Supabase |
| Incident response | Partial | No formal plan | Document + train |

### Recommendations

**Immediate (Week 1):**
- Fix 3 critical vulns
- Add HTTPS enforcement
- Document incident response

**Short-term (Month 1):**
- Implement user data download
- Add consent flows
- DPA with all vendors

**Long-term (Month 3):**
- Implement user deletion pipeline
- Regular security training
- Annual audit cycle

### Next Steps
1. Assign owner for each finding
2. Schedule weekly sync to track progress
3. Re-audit in 3 months

---
Report prepared by: Lein-Labz Security Team
Date: March 18, 2026
Confidential
```

---

## Fase 5: Produtos Digitais (Dias 14-30)

### Catálogo de Produtos

**Ideia:** Vender templates, courses, assets que reduzem tempo de desenvolvimento para outros builders.

#### Produto 1: N8N Workflows Bundle (R$ 199)
- 10 workflows prontos (WhatsApp, email, CRM, integrations)
- Importáveis direto em n8n
- Documentação em PT-BR
- Suporte via Discord

#### Produto 2: Supabase Multi-Tenant Template (R$ 299)
- Next.js + Supabase starter
- RLS policies pré-configuradas
- Auth flow completo
- Stripe integration
- GitHub repo privado

#### Produto 3: "LGPD Compliance Checklist" (R$ 99)
- Spreadsheet interativo (Google Sheets)
- 50+ itens LGPD
- Tracking de progresso
- Legal templates (privacy policy, DPA)

#### Produto 4: "WhatsApp SaaS em 30 Dias" Course (R$ 497)
- 15 video aulas (PT-BR)
- Arquivo do projeto final
- Comunidade Discord
- Lifetime access

#### Produto 5: LI Design System (R$ 149)
- Figma file: components, colors, typography
- 100+ components (buttons, forms, modals, etc.)
- Responsive design
- Dark mode included

---

### Pesquisa: Plataformas de Venda (Hotmart, Gumroad, Stripe)

**Baseado em conhecimento (sem web search específica):**

#### Hotmart (Brasil)
- **Vantagem:** Pagamento em PIX, Boleto (Brasil native)
- **Comissão:** 10-20% (por product)
- **Ideal para:** Courses, e-books, software downloads
- **Exemplo:** LI Course "WhatsApp SaaS em 30 Dias" → R$ 497 × 0.85 comissão = R$ 422

#### Gumroad (USA)
- **Vantagem:** Simples, payout direto, sem burocracia
- **Comissão:** 10% (standard)
- **Ideal para:** Digital assets, tools, one-off products
- **Exemplo:** Workflows Bundle → R$ 199 × 0.90 = R$ 179

#### Stripe Direct
- **Vantagem:** Máximo controle, branding próprio
- **Comissão:** 2.7% + R$ 0.30 (cartão), ~2% PIX
- **Ideal para:** SaaS, subscriptions, owned audience
- **Exemplo:** WhatsApp SaaS → R$ 299/mês × 0.98 = R$ 293/mês

**Recomendação:** Usar TODOS os 3
- Hotmart: Courses (Brasil)
- Gumroad: Templates/assets (global)
- Stripe: SaaS + proprietary (LI Hub)

---

### Pesquisa: Templates SaaS que Vendem

#### Characteristics of Best-Selling Templates

1. **Solve specific problem:** Não "general SaaS template", sim "WhatsApp Chatbot Template"
2. **Include documentation:** Video walkthrough min. 3-5 mins
3. **Codebase familiar:** React + Next.js (que devs usam)
4. **Incluir exemplos:** Database schema pronto, 3+ sample workflows
5. **Suporte:** Discord community ou email

#### Top Selling Categories (Global)

```
1. AI Tools ($297-2000)
   - ChatGPT wrapper templates
   - Prompt builder tools

2. SaaS Starters ($297-997)
   - Next.js + Stripe + Auth
   - No-code tool clones

3. Course Platforms ($397-1497)
   - Teachable alternative
   - Learning management

4. Landing Page Builders ($99-297)
   - Framer templates
   - Webflow templates

5. Content Monetization ($197-597)
   - Newsletter tools
   - Gumroad alternatives
```

#### Lein-Labz Product Positioning

**Mercado:** Builders brasileiros (devs, agências, consultores)
**Budget:** R$ 100-500 (maior que global, mas menor que USA)
**Problema:** "Quero monetizar, mas não sei como estruturar"

**Estratégia:** Bundle approach
```
Starter Bundle (R$ 99):
  - LGPD Checklist
  - Supabase RLS boilerplate

Pro Bundle (R$ 299):
  - N8N Workflows x10
  - Supabase template (full)

Agency Bundle (R$ 597):
  - Tudo acima
  + LI Design System (Figma)
  + WhatsApp SaaS source code
```

---

### Estratégia de Lançamento (Dias 14-30)

#### Week 1 (Dias 14-18): Criação + Landing Page

- [ ] Finalize produto (workflows, docs, assets)
- [ ] Cria landing page (Framer, Webflow ou Next.js)
- [ ] Produz demo video (5 mins, PT-BR)
- [ ] Setup pagamento (Hotmart para courses, Gumroad para assets)

#### Week 2 (Dias 19-24): Marketing + Soft Launch

**Content Marketing:**
- LinkedIn posts (3x semana): behind-the-scenes, learnings
- Twitter: "Building in public" thread
- Blog posts (2-3): How I built WhatsApp SaaS, LGPD checklist, n8n patterns

**Email Campaign:**
- Send segmented (to previous users / newsletter)
- Subject: "New: WhatsApp SaaS Template ($299)"
- Copy: pain point → demo → CTA

**Community Marketing:**
- Post em Indie Hackers
- Dev.to artigos
- Facebook groups Brasil (WebDevelopers, AI Builders)

#### Week 3 (Dias 25-30): Launch + Ads

**Launch Day:**
- Product Hunt submission (if global appeal)
- Twitter thread with demo
- Email blast to newsletter
- Announcement in Discord

**Paid Ads (se houver budget):**
- Facebook/Instagram: Targeted to web devs, agencies (R$ 500-1000 budget)
- Google Ads: Keywords "n8n template", "whatsapp saas", "lgpd brasil"

#### Metricas de Sucesso

- Week 1-2: 100 visits, 10 conversions = 10% conversion
- Week 2-3: 500 visits, 75 conversions = 15% conversion
- Week 4: 1,000 visits, 200 conversions = 20% conversion

**Revenue Projection:**
- 50 courses × R$ 497 = R$ 24,850
- 30 templates × R$ 299 = R$ 8,970
- 10 design systems × R$ 149 = R$ 1,490
- **Total Month 1:** ~R$ 35k (squad gets 50% = R$ 17,500)

---

## Fase 6: Conteúdo e LGPD Consulting (Dias 21-30)

### Monetização de Conteúdo

#### Content Strategy

**Pillar 1: Blog (thelenilabz.com/blog)**
- Articles: How to build WhatsApp SaaS, LGPD compliance, n8n patterns
- SEO: Target long-tail keywords ("como monetizar whatsapp", "lgpd compliance saas")
- Monetization: Affiliate links (Hotmart, Stripe), lead gen (email)

**Pillar 2: Newsletter ("Builders Brasil")**
- Frequência: Weekly
- Conteúdo: Tools, trends, tutorials, product launches
- Monetization: Sponsored sections (R$ 500/week), affiliate, premium tier (R$ 9/mês)

**Pillar 3: YouTube**
- Serie: "Building Lein-Labz" (documento journey)
- Series: n8n tutorials, Supabase + RLS, WhatsApp integrations
- Monetization: Ad revenue (YouTube Partner), affiliate, course links

**Pillar 4: Twitter / LinkedIn**
- Daily tips, insights, launches
- Thread format: tutorials, industry analysis
- Monetization: Sponsorships, lead gen

---

### Consultoria LGPD como Serviço

#### Service Offering

**LGPD Audit + Consulting**
- Preço: R$ 5,000 - 15,000 (por projeto)
- Escopo: 2 semanas de trabalho
- Deliverable: Audit report + roadmap

**LGPD Compliance Implementation**
- Preço: R$ 10,000 - 30,000 (retainer 3 meses)
- Escopo: Implementar recomendações do audit
- Deliverable: Weekly check-ins, docs, policies

**Expert Review (Ad-hoc)**
- Preço: R$ 1,500/hora
- Escopo: Consultoria jurídica-técnica
- Exemplo: "How to delete user data legally?" = 2h = R$ 3k

#### Sales Playbook

**Alvo:** CTOs, founders de startups Series A/B, CPOs (Chief Privacy Officers)

**Outreach Template:**
```
"Oi [Nome], você está preparando sua startup para Series A, certo?
Série A exige compliance LGPD + security audit.

A gente faz isso: LGPD audit em 2 semanas, relatório executivo + roadmap.
Mais barato que contratar consultoria jurídica, mais rápido que build interno.

Posso fazer call rápida pra entender se faz sentido?

Abraços,
[Squad]"
```

**Expected Close Rate:** 10% (1-2 clientes/mês)
**ARR Goal:** 5 retainers × R$ 30k = R$ 150k (squad split = R$ 75k)

---

## Referências Técnicas

### Claude MCP – Capacidades por Ferramenta

**Fonte:** [Claude Code MCP Integration Guide](https://vladimirsiedykh.com/blog/claude-code-mcp-workflow-playwright-supabase-figma-linear-integration-2025), [Supabase MCP Docs](https://supabase.com/docs/guides/getting-started/mcp)

#### Linear MCP

**Capabilities:**
- Create/update issues programmatically
- Assign issues to team members
- Add labels, priorities, milestones
- Automatically create issues from Claude analysis
- Query Linear database

**Use Case - Lein-Labz:**
```
Claude reads GitHub PR → Creates Linear issue for QA testing
Supabase schema change → Auto-creates Linear subtask
Customer feedback → Claude files bug in Linear (with priority)
```

#### Supabase MCP

**Capabilities:**
- Design tables and schema
- Generate migrations
- Query database (SELECT/INSERT/UPDATE)
- Manage auth configuration
- Deploy edge functions
- Generate TypeScript types

**Use Case - Lein-Labz:**
```
Squad briefs Claude: "We need multi-tenant auth"
Claude designs schema, creates RLS policies, generates migrations
Claude generates TypeScript types for frontend
All from natural language in Claude Code
```

#### Notion MCP

**Capabilities:**
- Read/write Notion pages
- Query databases
- Create docs from Claude analysis
- Push reports to Notion

**Use Case - Lein-Labz:**
```
Audit tool generates findings → Claude pushes to Notion doc
Daily standups in Notion → Claude summarizes for stakeholders
Product roadmap in Notion → Claude updates based on feedback
```

#### How to Setup

```bash
# Linear
claude mcp add --name linear linear

# Supabase
claude mcp add --name supabase supabase

# Notion
claude mcp add --transport http notion https://mcp.notion.com/mcp
```

---

### Padrões de Código do Squad

#### Naming Conventions

**Branches:**
```
feature/whatsapp-webhook
fix/rls-policy-tenant-id
docs/lgpd-checklist
hotfix/stripe-webhook-timeout
```

**Commits:**
```
feat: add multi-tenant support to chats table
fix: RLS policy blocker users from other tenants
docs: LGPD compliance checklist updated
refactor: extract webhook validation to lib/validate.ts
test: add tests for Evolution API webhook
```

**Files:**
```
lib/supabase.ts (client utilities)
lib/stripe.ts (stripe helpers)
types/database.ts (auto-generated, DON'T EDIT)
components/ChatUI.tsx (React components)
pages/api/webhooks/evolution.ts (API routes)
```

#### Code Organization

```
apps/whatsapp-saas/
├─ src/
│  ├─ components/     (React components)
│  ├─ pages/          (Next.js pages)
│  ├─ api/            (API routes + webhooks)
│  ├─ lib/            (utilities, helpers)
│  ├─ types/          (TypeScript types)
│  └─ styles/         (CSS/Tailwind)
├─ public/             (static assets)
└─ tests/              (Jest tests)

packages/
├─ db/                 (Supabase schemas, migrations)
├─ ui/                 (shared React components)
└─ lib/                (shared utilities)
```

#### Key Files to Maintain

**Database Migrations:**
- File: `packages/db/supabase/migrations/[timestamp]_create_table.sql`
- Always include RLS policies in migration
- Never drop columns (add nullable columns instead)

**Environment Variables:**
- File: `.env.local.example` (commit this, never `.env.local`)
- All secrets in Vercel dashboard or `.env.local`

**API Documentation:**
- File: `docs/api.md`
- Document each endpoint: path, method, auth, payload, response

---

### Convenções de Commit e Branch

#### Branch Strategy (Git Flow Simplified)

```
main
  ↑ (PR from develop)
develop
  ↑ (PR from feature branches)
feature/[feature-name]
fix/[bug-name]
hotfix/[critical-issue] (from main only)
```

#### Commit Message Convention

```
<type>(<scope>): <subject>

<body>

<footer>
```

**Type:** feat, fix, docs, style, refactor, test, chore
**Scope:** whatsapp, stripe, auth, rls, etc.
**Subject:** Imperative, lowercase, <50 chars

**Example:**
```
feat(whatsapp): add message scheduler for bulk send

Implement scheduled messaging for campaigns. Users can:
- Create schedule (date/time)
- Target by segment (tag, language)
- Preview before send
- Receive notification when sent

Closes #142
```

#### PR Requirements

- [ ] Tests pass (GitHub Actions)
- [ ] Code reviewed (1 approval min)
- [ ] No conflicts with develop
- [ ] Commit messages follow convention
- [ ] Security check passed (Semgrep, Grype)

---

### Segurança e Boas Práticas

#### Secrets Management

**NUNCA committar:**
- API keys (Stripe, Supabase, etc.)
- Passwords, tokens
- Private URLs
- Database credentials

**Para guardar secrets:**
1. Vercel Environment Variables (production)
2. `.env.local` (development, in .gitignore)
3. Bitwarden / 1Password (team sharing)
4. Supabase Vault (edge functions)

#### Data Protection (LGPD)

**Minimum Checklist:**
- [ ] Document what data you collect (data inventory)
- [ ] Have clear consent (privacy modal, opt-in)
- [ ] Implement user access endpoint (`GET /api/user/export`)
- [ ] Implement user deletion (`DELETE /api/user`)
- [ ] Hash passwords (bcrypt minimum 10 rounds)
- [ ] Encrypt sensitive data at rest (PII, payment info)
- [ ] Log access to sensitive data (audit trail)
- [ ] Have incident response plan

**Encryption Recommendation:**
```typescript
// At rest: Supabase handles TLS + physical security
// In transit: HTTPS always (Vercel enforces)
// In database: Encrypt PII fields manually if needed
import crypto from 'crypto';

const encryptPII = (plaintext: string, key: string) => {
  const iv = crypto.randomBytes(16);
  const cipher = crypto.createCipheriv('aes-256-cbc', Buffer.from(key), iv);
  return iv.toString('hex') + ':' + cipher.update(plaintext).toString('hex');
};
```

#### Rate Limiting

**Evolution API:**
- Max 1,000 messages/minute
- Implement queue in n8n

**Stripe API:**
- Max 100 requests/second
- SDK handles rate limiting

**Custom API (Next.js):**
```typescript
// Middleware: Rate limit by IP
import rateLimit from 'express-rate-limit';

const limiter = rateLimit({
  windowMs: 60 * 1000, // 1 minute
  max: 100, // 100 requests per minute
});

export default limiter;
```

#### Monitoring + Alerting

**Tools:**
- Sentry (errors)
- PostHog (analytics)
- Uptime Robot (status page)

**What to Alert On:**
- Webhook failures (Stripe, Evolution)
- Database connection errors
- API error rate > 5%
- Unauthorized access attempts (401/403 > 10 in 5 min)

---

## Conclusão

Lein-Labz é um experimento intenso de construção de 4 negócios paralelos em 30 dias. O sucesso depende de:

1. **Execução rápida:** MVP → feedback → iterate (weekly cycles)
2. **Foco:** Cada Squad member owned 1 produto principal
3. **Automação total:** Use n8n, Claude, Supabase ao máximo; evite código custom desnecessário
4. **LGPD desde dia 1:** Não é "depois"; é competitivo diferencial
5. **Sales early:** Venda antes de build estar perfeito; feedback direto de paying customers é ouro

**Ramp-up Financeiro (Projeção Mês 1-3):**
```
Mês 1: R$ 30-40k (primeiras vendas, agência projects)
Mês 2: R$ 80-120k (WhatsApp SaaS traction, retainers)
Mês 3: R$ 150-250k (3 produtos em produção, consulting)
```

**Próximas Ações (Antes de Dia 1):**
- [ ] Setup GitHub org + Vercel projects
- [ ] Criar conta Supabase project
- [ ] Stripe account Brasil
- [ ] n8n self-hosted (Render)
- [ ] Divisão de tarefas por Squad member
- [ ] Daily standup: 9am PT (ou timezone do squad)

---

## Recursos Externos (Web Search Sources)

### Evolution API & WhatsApp
- [Evolution API Documentation](https://doc.evolution-api.com/v2/en/integrations/cloudapi)
- [GitHub - EvolutionAPI](https://github.com/EvolutionAPI/evolution-api)
- [Gurusup - Evolution API 2026](https://gurusup.com/blog/evolution-api-whatsapp)

### n8n Automations
- [n8n Workflows Community - 8515 templates](https://n8n.io/workflows/)
- [GitHub - awesome-n8n-templates](https://github.com/enescingoz/awesome-n8n-templates)

### Supabase RLS & Auth
- [Supabase Docs - Row Level Security](https://supabase.com/docs/guides/database/postgres/row-level-security)
- [Supabase - Multi-tenancy Examples](https://supabase.com/docs/guides/realtime/examples/multi-tenancy)

### Vercel & Next.js
- [Vercel - Environment Variables](https://vercel.com/docs/environment-variables)
- [Next.js - Deployment Guide](https://nextjs.org/docs/app/getting-started/deploying)

### Stripe Brasil
- [Stripe - PIX Payments](https://docs.stripe.com/payments/pix)
- [Stripe Brasil Pricing](https://stripe.com/en-br/pricing)

### LGPD Compliance
- [Planalto - Lei 13709](https://www.planalto.gov.br/ccivil_03/_ato2015-2018/2018/lei/l13709.htm)
- [Iugu Blog - LGPD para SaaS](https://www.iugu.com/blog/lgpd-para-saas)

---

**Document Version:** 1.0
**Created:** March 18, 2026
**Status:** Active - Lein-Labz Squad Reference
**Last Updated:** March 18, 2026