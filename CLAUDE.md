# CLAUDE.md - Lein-Labz Squad FIAP

## 📋 Contexto do Projeto

**Nome do Projeto:** Lein-Labz (Squad FIAP)

**Membros da Equipe:**
- **Ricardo** - Agentes de IA, estratégia geral e liderança
- **Luís** - Sistemas complexos, alta disponibilidade e infra
- **Nicolas** - Full-stack, integrações e DevOps
- **Henrique** - Cibersegurança, LGPD e IA aplicada

**Duração:** 30 dias de sprint intensivo para construir 6 projetos de negócio

**Capital Inicial:** R$500-2000
**Meta de Receita:** R$15k-50k/mês ao final do período

---

## 💰 Projetos de Negócio (Rankeados por Retorno Financeiro)

### 1. **Agência de Automação com IA**
- **Receita Esperada:** R$8k-30k/mês
- **Modelo:** Serviços de automação de processos usando n8n + Claude
- **Público-alvo:** PMEs e startups
- **Entregáveis:** Fluxos automáticos, dashboards, integrações
- **Validação:** Estudos de caso de clientes já em progresso
### 2. **WhatsApp SaaS (Micro-SaaS com Evolution API)**
- **Receita Esperada:** R$5k-25k/mês
- **Modelo:** SaaS de recurringência (planos mensais)
- **Funcionalidades:** Gestão de conversas, automação, APIs
- **Público-alvo:** Pequenos negócios e startups
- **Validação:** MVP em produção em semana 1

### 3. **CyberShield Security**
- **Receita Esperada:** R$3k-12k/mês
- **Modelo:** Auditorias de segurança + consultoria LGPD
- **Entregáveis:** Relatórios de compliance, remediação
- **Público-alvo:** Empresas que precisam de conformidade
- **Lead Time:** 2-3 semanas por projeto

### 4. **Produtos Digitais**
- **Receita Esperada:** R$1k-8k/mês
- **Modelo:** Venda de templates, boilerplates, packs n8n
- **Plataformas:** Gumroad, Lemonsqueezy
- **Público-alvo:** Devs, creators, empreendedores
- **Validação:** Lançamento em semana 2

### 5. **Conteúdo Monetizado**
- **Receita Esperada:** R$500-5k/mês
- **Modelo:** YouTube, blog, cursos e webinars
- **Conteúdo:** Tutoriais, case studies, guias de IA
- **Públicos-alvo:** Devs, empreendedores, CFOs
- **Validação:** Publicações iniciadas em semana 1

### 6. **Consultoria LGPD**
- **Receita Esperada:** R$2k-10k/mês
- **Modelo:** Consultoria especializada em compliance
- **Entregáveis:** Auditorias, políticas, treinamentos
- **Público-alvo:** Empresas sob regulação LGPD
- **Lead Time:** 1-2 semanas de escopo
---

## 🛠️ Tech Stack

**Frontend:**
- Next.js 14+ (App Router)
- Tailwind CSS
- shadcn/ui (componentes)
- TypeScript

**Backend:**
- Supabase (autenticação, banco de dados, storage, edge functions)
- PostgreSQL
- Realtime (se necessário)

**Deploy & Hosting:**
- Vercel (frontend, funções serverless)
- Supabase Cloud (banco de dados)
- n8n Cloud (automações)

**Pagamentos:**
- Stripe (checkout, webhooks, recurring)
- Stripe Atlas (futuro - estrutura empresarial)

**Automações:**
- n8n (orquestração de workflows)
- Evolution API (WhatsApp)
- Automações CLI com Claude

**IA & Integração:**
- Claude Max (2-3 planos de assinatura)
- Claude API para automações
- MCP (Model Context Protocol)
---

## 🗂️ Ferramentas de Organização (Integradas com Claude via MCP)

### Linear
**Propósito:** Gestão centralizada de tarefas

**Estrutura de Times:**
1. **Squad Ops** - Gestão, financeiro, legal
2. **Agência IA** - Projeto de automação e serviços
3. **WhatsApp SaaS** - Desenvolvimento e crescimento
4. **CyberShield** - Auditorias e compliance
5. **Produtos Digitais** - Templates e boilerplates

**Rótulos (Labels):**
- `revenue` (verde) - Gera receita direto
- `product` (azul) - Desenvolvimento de produtos
- `infra` (roxo) - Infraestrutura e devops
- `blocked` (amarelo) - Bloqueante
- `quick-win` (ciano) - Rápido e impactante
- `bug` (vermelho)
- `documentation` (cinza)
**Prioridades:**
- Urgent (RED) - Revenue-labeled sempre
- High (ORANGE)
- Medium (BLUE)
- Low (GRAY)

### Notion
**Propósito:** Base de conhecimento, documentação, financeiro

**Seções Principais:**
- Estratégia geral e roadmap
- Documentação técnica
- Contratos e propostas (template)
- Notas de reunião
- Planilhas financeiras
- Case studies de clientes
- Wiki de processos

**Regra:** Tarefas NÃO entram em Notion. Tarefas vão SEMPRE para Linear.

### GitHub
**Propósito:** Controle de versão e code review

**Organização:** `ricardopaivamelo/Lein-Labz`

**Repositórios por Projeto:**
- `lein-labz-website` (site principal)
- `agencia-ia-tools` (ferramentas automação)
- `whatsapp-saas` (plataforma WhatsApp)
- `cybershield-audit` (ferramenta CyberShield)
- `digital-products` (templates e packs)
### n8n
**Propósito:** Automação de workflows internos e clientes

**Workflows Críticos:**
- **Daily Report** (09:00) - Status consolidado para Linear/Slack
- **Lead Notification** (em tempo real) - Novos leads via WhatsApp
- **Revenue Tracker** (diária) - Atualiza Stripe → Notion
- **Deploy Monitor** (24/7) - Monitora Vercel → Slack

### Google Drive
**Propósito:** Assets visuais, propostas, contratos

**Estrutura:**
- `Lein-Labz/` (pasta raiz)
  - `Propostas/` - Templates de proposta
  - `Contratos/` - Modelos legais
  - `Assets/` - Logos, banners, thumbnails
  - `Financeiro/` - Planilhas compartilhadas

---

## 🌿 Git Workflow

**Modelo:** Trunk-based development com proteção de main

**Branches:**
- `main` - Sempre pronta para produção (production)
- `feature/xxx` - Branches de feature (máximo 1-2 dias de vida)
- `hotfix/xxx` - Correções críticas em produção
**Regras:**
- Pull Request obrigatória para main
- Mínimo 1 aprovação antes de merge
- CI/CD roda automaticamente via GitHub Actions
- Toda PR deve linkar issue no Linear

**Convenção de Commits:**
```
feat: descrição breve da feature
fix: descrição do bug corrigido
docs: atualização de documentação
chore: tarefas de manutenção
refactor: reorganização de código
test: testes ou cobertura
```

**Integração Linear-GitHub:**
- Issue Linear linkada em PR → Auto-fecha ao fazer merge em main
- GitHub Actions dispara ao fazer push em feature/*

---

## 📁 Estrutura do Repositório

```
Lein-Labz/
│
├── public/                    # Site estático e documentação pública
│   ├── index.html            # Hub central - links para todos os docs
│   ├── estrategia.html       # Documento de estratégia 30 dias
│   ├── organizacao.html      # Sistema de organização do squad
│   ├── roadmap.html          # Roadmap visual dos 6 projetos
│   └── assets/               # Logos, favicons, imagens│
├── apps/                      # Aplicações principais
│   ├── website/              # Site marketing
│   │   ├── app/
│   │   ├── components/
│   │   └── public/
│   │
│   ├── agencia-ia/           # Dashboard Agência de IA
│   │   ├── app/
│   │   ├── lib/
│   │   └── supabase/
│   │
│   ├── whatsapp-saas/        # Plataforma WhatsApp
│   │   ├── app/
│   │   ├── api/
│   │   └── lib/
│   │
│   └── cybershield/          # Dashboard CyberShield
│       ├── app/
│       ├── lib/
│       └── reports/
│
├── packages/                  # Pacotes compartilhados
│   ├── ui/                   # Componentes shadcn
│   ├── types/                # TypeScript types globais
│   └── utils/                # Utilitários compartilhados
│
├── supabase/                  # Migrations, RLS policies
│   ├── migrations/
│   ├── functions/            # Edge functions
│   └── policies/│
├── n8n/                       # Definições de workflows
│   ├── workflows/
│   ├── templates/
│   └── credentials.example.json
│
├── .github/
│   └── workflows/             # GitHub Actions CI/CD
│
├── CLAUDE.md                  # Este arquivo
├── SKILL.md                   # Skill abrangente para Claude
├── README.md                  # Documentação pública
├── package.json
├── turbo.json                 # Se usando monorepo com Turbo
├── vercel.json                # Configuração Vercel
└── .env.example               # Variáveis de ambiente
```

---

## 📋 Convenções Principais

**Linguagem:** Português (Brasil) para toda comunicação, docs e código (comentários)

**Gestão de Tarefas:**
- ✅ Tarefas vão SEMPRE em Linear, NUNCA em Notion
- Notion é exclusivamente para knowledge base e documentação
- Cada task em Linear deve ter: assignee, label, milestone

**Sprints:**
- **Duração:** Segunda a sexta (5 dias úteis)
- **Limite:** 5-7 issues por membro por sprint
- **Review:** Sextas à tarde
**Reuniões:**
- **Daily Standup:** 09:00 via chamada WhatsApp (15 min máximo)
  - O que fiz? O que vou fazer? Bloqueantes?
- **Sprint Review:** Sexta 15:00 no Linear
- **Bloco de Vendas:** 18:00 todos os dias (não pode schedule de dev)

**Priorização de Revenue:**
- Issues com label `revenue` recebem **SEMPRE** prioridade "Urgent"
- Revenue é avaliado diariamente
- Métricas: MRR, ARR, CAC, LTV

---

## 🎯 Estratégia: "Cash Cow + Moonshot"

**Eixo 1 - Cash Cow (Receita Imediata):**
- Agência de Automação com IA
- Consultoria LGPD
- CyberShield (serviços)
- Objetivo: R$10k-40k/mês em 30 dias

**Eixo 2 - Moonshot (Recurring/Escalável):**
- WhatsApp SaaS
- Produtos Digitais
- Conteúdo Monetizado
- Objetivo: Base de R$5k-10k/mês escalável

**Regra de Ouro:**
```
Nunca gaste mais de 3 dias buildando sem validação.
Validação = Feedback de cliente real ou teste do produto.
Pivote rápido se validação falhar.
```

**Processo de Validação:**
1. MVP em 1-2 dias
2. Teste com 3-5 clientes ou prospects reais
3. Feedback direto (survey, call, Discord)
4. Iterate ou pivote em máximo 2 dias
5. Go-to-market se passar validação
---

## 💹 Cenários Financeiros

### Mês 1 (Sprint atual)
- **Conservador:** R$2,400/mês (primeiros clientes)
- **Realista:** R$8,500/mês (2-3 agência + 1 SaaS)
- **Otimista:** R$18,000/mês (tração em todos)

### Mês 2 (Projeção)
- **Conservador:** R$14,400/mês
- **Realista:** R$34,500/mês
- **Otimista:** R$66,000/mês

### Mês 3+ (Sustentação)
- Foco em retenção de recurring
- Escala do Eixo 1 via marketing
- Produto de Moonshot já gerando receita

**Tracking:**
- Dashboard de receita em Notion (atualizado diariamente por n8n)
- Stripe → Google Sheets → Notion automaticamente
- Review financeiro toda segunda-feira 08:00

---

## 🤖 Diretrizes de Uso do Claude

**Quando Usar Claude:**
- Geração de código (templates, boilerplates)
- Code review de PRs complexas
- Documentação técnica
- Relatórios financeiros automáticos
- Briefings diários
- Criação de issues em Linear (via script)
- Queries SQL em Supabase
- Análise de dados de Stripe
- Criação de propostas e contratos
**Capacidades Principais:**
- Criar/atualizar issues em Linear via API
- Buscar documentos em Notion
- Executar queries em Supabase via SQL
- Verificar status de deploys em Vercel
- Gerenciar transações via Stripe API
- Analisar logs de n8n
- Gerar reports de performance

**Skills Dedicadas:**
- 1 Skill por projeto (Agência, WhatsApp, CyberShield, etc.)
- 1 Skill de Ops (financeiro, legal, reporting)
- 1 Skill de Infra (Supabase, Vercel, n8n)
- Todas com contexto persistente via embeddings

**Limitações Conscientes:**
- Claude não cria contas ou entra em sistemas
- Claude executa código conforme aprovado
- Claude não faz decisões de negócio (recomenda apenas)
- Claude não acessa dados sensíveis sem MCP integrado
- Senha/tokens NUNCA passados em prompt

---

## 🔐 Segurança & Compliance

**LGPD (Lei Geral de Proteção de Dados):**
- Privacy by default em todos os projetos
- Consentimento explícito para coleta de dados
- Direito ao esquecimento implementado
- Policies de retenção configuradas

**Cibersegurança:**
- Secrets gerenciados via Vercel/Supabase
- `.env` local nunca commitado
- 2FA em GitHub, Linear, Stripe
- Auditoria de acesso mensal
- Bug bounty para clientes de CyberShield
**Code Security:**
- Dependabot habilitado (GitHub)
- npm audit rodando em CI
- OWASP Top 10 checklist
- SQL injection protection (Supabase RLS)

---

## 📞 Contato & Escalation

**Comunicação Principal:** WhatsApp Squad
**Issues/Tasks:** Linear
**Documentação:** Notion
**Code:** GitHub + Discord tech channel
**Financeiro:** Planilha Notion (semanal)

**Escalação Rápida:**
- Bloqueante de receita → Ricardo imediatamente
- Down de produção → Luís/Nicolas em tempo real
- Security issue → Henrique prioridade máxima
- Customer complaint → Projeto lead

---

## 📅 Timeline (30 dias)

**Semana 1:** MVPs de todos os 6 projetos + validação
**Semana 2:** Primeiro cliente em cada projeto + refinamento
**Semana 3:** Escala do Cash Cow + otimizações
**Semana 4:** Review, análise, planning Month 2 + célula de crescimento

**Milestones:**
- Day 3: 1º cliente Agência IA
- Day 5: WhatsApp SaaS live
- Day 7: CyberShield primeira auditoria
- Day 10: Produtos Digitais lançados
- Day 14: R$8k/mês em receita
- Day 30: R$15k+/mês em receita
---

## ✅ Checklist de Setup

- [ ] Repositório GitHub criado e organizações configuradas
- [ ] Linear teams e labels criadas
- [ ] Notion workspaces com templates
- [ ] Supabase project criado
- [ ] Vercel projects linkados
- [ ] Stripe account com produtos
- [ ] n8n workflows de reporting
- [ ] Google Drive folders estruturados
- [ ] Claude Max subscriptions (2-3 planos)
- [ ] Primeiras propostas de clientes prontas
- [ ] Daily standup agendado (recorrente)
- [ ] SKILL.md criada para Claude
- [ ] MCP integrations testadas

---

**Versão:** 1.0
**Última Atualização:** 2026-03-18
**Status:** 🟢 Ativo - Sprint 1 em andamento