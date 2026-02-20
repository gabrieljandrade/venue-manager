# Venue Manager — Projetor Financeiro de Shows e Turnês

Ferramenta de projeção financeira para produtores e gestores de turnês musicais. Permite simular receitas, custos e resultado líquido de cada show antes de fechar contratos.

---

## Contexto e Problema

A produção de uma turnê envolve dezenas de variáveis financeiras que precisam ser estimadas antes mesmo de qualquer show acontecer: cachê do artista, locação de venue, equipamentos técnicos, logística, impostos, divisão de ingressos por tipo (inteira, meia, solidário), capacidade por setor, preços por setor...

Antes dessa ferramenta, esse trabalho era feito em planilhas — uma por show, difíceis de atualizar, propensas a erros e sem visão consolidada da turnê inteira.

O **Venue Manager** centraliza isso: você cria um orçamento, adiciona os shows da turnê, ajusta os parâmetros de cada um, e tem a DRE (Demonstração de Resultado) por show e consolidada em tempo real.

---

## Funcionalidades

### Orçamentos e Shows
- Criar múltiplos orçamentos (ex: turnê 2026, turnê alternativa, cenário conservador)
- Cada orçamento contém N shows
- Cada show é vinculado a um venue com dados pré-preenchidos (capacidade, custos padrão, impostos)
- Persistência automática no `localStorage` do navegador

### Venues Pré-cadastrados
12 venues brasileiros com setores, preços e custos reais já configurados:
- Espaço Unimed (SP), Vivo Rio (RJ), BeFly Hall (BH), Araújo Viana (POA), Ópera de Arame (CWB), Cine Teatro São Luiz (FOR), Teatro Castro Alves (SSA), Espaço Ibirapuera (SP), Teatro Guaíra (CWB), Teatro Amazonas (MAO), Teatro da Paz (BEL), Arena Jockey (RJ)

### Distribuição de Ingressos
- Três tipos: **Meia Entrada** (50% do preço inteira), **Solidário** (% configurável, padrão 67,5%), **Inteira** (100%)
- Parâmetros de distribuição por setor (% de meia / solidário / inteira)
- Aplicação em lote: define o split e aplica a todos os setores de uma vez
- **Solidário como % da Inteira** configurável por show — altera automaticamente o preço solidário em todos os setores

### Precificação por Setor
- Cada setor tem preço próprio
- Ao alterar o **Preço Inteira**, Meia e Solidário se recalculam automaticamente
- Capacidade editável por setor (campo âmbar) — ao alterar, recalcula as quantidades pelo split atual
- Possibilidade de adicionar novos setores manualmente

### Custos de Produção
- Organizados em categorias: **Deduções Legais** (ECAD, ISS+PIS), **Venue** (Locação), **Produção Artística** (Produção, Cachê), **Técnica** (Backline, Iluminação, Sonorização, LED/Telas), **Logística**, **Outros**
- Custos pré-preenchidos pelo template do venue — editáveis diretamente por show
- Campos mostram o valor atual (template ou override); destaque azul indica que foi sobrescrito; botão ↺ restaura ao template
- Locação configurável por show: **Template** (usa o padrão do venue), **Valor Fixo** (R$ absoluto) ou **% da Receita** (percentual da receita bruta)
- Linhas de custo adicionais por categoria — com descrição e valor livres

### DRE em Tempo Real
- Painel lateral direito com DRE do show ativo atualizada a cada mudança
- Receita bruta, custos por categoria, resultado, margem e **ticket médio**
- DRE Consolidada da turnê inteira (acessível pelo botão no topo)
- KPI bar: Receita Total, Total Custos, Resultado, Ticket Médio consolidados

### Venues Customizados
- Criar venues novos com nome, cidade, estado, setores, custos padrão e alíquotas de impostos
- Venues customizados são salvos localmente e aparecem ao lado dos venues padrão
- Podem ser excluídos a qualquer momento

### Export Excel
- Botão disponível no Dashboard e na DRE Consolidada
- Gera um `.xlsx` com a DRE Consolidada + uma aba por show com o detalhamento por setor

---

## Arquitetura Técnica

### Por que um único arquivo HTML?

O app é um **Single-File React App** — todo o código (HTML, CSS, JavaScript/JSX) está em um único `index.html`. Essa decisão foi tomada por:

- **Deploy instantâneo**: basta hospedar um arquivo estático — sem build, sem servidor, sem processo de instalação
- **Portabilidade**: o arquivo pode ser aberto diretamente no navegador (sem servidor), compartilhado por e-mail ou hospedado em qualquer CDN
- **Sem dependências locais**: todas as libs são carregadas por CDN em runtime

### Dependências (CDN)

| Lib | Versão | Uso |
|-----|--------|-----|
| React | 18 | UI e state management |
| ReactDOM | 18 | Renderização no DOM |
| Babel Standalone | latest | Transpilação de JSX no browser |
| Tailwind CSS | CDN | Estilização utilitária |
| SheetJS (XLSX) | 0.18.5 | Geração do arquivo Excel |

> ⚠️ **Nota de performance**: O Babel Standalone faz a transpilação JSX no browser em runtime. Isso é aceitável para uma ferramenta interna/pequena equipe, mas adiciona ~2-3s de carregamento inicial. Para escalar para muitos usuários simultâneos, o ideal é migrar para um build com Vite + React.

### Persistência de Dados

Tudo é salvo no `localStorage` do navegador do usuário:
- `vm_budgets` — array de orçamentos
- `vm_custom_venues` — array de venues customizados

**Implicações:**
- Dados são locais por máquina e por navegador — não há sync entre dispositivos
- Ao limpar os dados do navegador, os orçamentos são perdidos
- Para colaboração em equipe, seria necessário um backend com banco de dados

---

## Estrutura do Código (`index.html`)

O arquivo está organizado em seções claramente delimitadas com comentários `// ─── NOME ───`:

```
index.html
│
├── <head>           → CDN imports, Tailwind config, estilos globais
│
└── <script type="text/babel">
    │
    ├── VENUE CONTEXT          → createContext para compartilhar venues entre componentes
    ├── STATIC VENUES          → Array VENUES[] com os 12 venues pré-cadastrados
    ├── COST CATEGORIES        → Arrays DRE_CATS e OVERRIDE_CATS (categorias de custo)
    ├── STORAGE                → Funções loadBudgets/saveBudgets/loadCustomVenues/saveCustomVenues
    ├── UTILS                  → uid(), fmtR(), fmtPct()
    ├── MODEL                  → Constantes padrão + funções defaultSectorQtys(), makeShow(), makeBudget()
    ├── CALCULATIONS           → calcSectorRevenue(), calcShow(), calcBudget()
    ├── EXCEL EXPORT           → exportToExcel()
    ├── SHARED UI              → NInput, DRERow (componentes primitivos)
    ├── DRE PANEL              → DREPanel (DRE lateral do show ativo)
    ├── SECTOR TABLE           → SectorTable (tabela de setores com preços e quantidades)
    ├── GLOBAL SPLITS EDITOR   → GlobalSplitsEditor (% meia/solidário/inteira + % solidário da inteira)
    ├── LOCAÇÃO EDITOR         → LocacaoEditor (toggle template/fixo/% receita)
    ├── CATEGORY COSTS EDITOR  → CatCostLines (adicionar linhas de custo por categoria)
    ├── SHOW FORM              → ShowForm (formulário completo de configuração do show)
    ├── CREATE VENUE MODAL     → CreateVenueModal (modal de criação de venue customizado)
    ├── DRE CONSOLIDADA        → DREConsolidated (modal com DRE da turnê toda)
    ├── EDITOR                 → Editor (tela principal de edição do orçamento)
    ├── DASHBOARD              → Dashboard (tela inicial com lista de orçamentos)
    └── APP ROOT               → App (raiz da aplicação, Provider do VenueCtx)
```

---

## Modelo de Dados

### Venue (template — não mutável pelo usuário)
```js
{
  id: 'espaço-unimed-sp',         // string, único
  name: 'Espaço Unimed',
  city: 'São Paulo',
  state: 'SP',
  totalCapacity: 3250,
  cortesias: 80,                  // número de ingressos cortesia (informativo)
  sectors: [
    {
      id: 'platinum',
      name: 'PLATINUM',
      capacity: 360,
      priceInteira: 500,          // preço base do setor (inteira = 100%)
      splitMeia: 0.4,             // % da capacidade para meia entrada
      splitSolidario: 0.5,        // % da capacidade para solidário
      splitInteira: 0.1,          // % da capacidade para inteira
    }
    // ...mais setores
  ],
  costs: {
    locacaoType: 'pct',           // 'pct' ou 'fixed'
    locacaoValue: 0.40,           // 0.40 = 40% da receita (se pct) | valor em R$ (se fixed)
    producao: 182690,
    logistica: 26400,
    backline: 10000,
    iluminacao: 30000,
    som: 30000,
    led: 30000,
    outros: 800,
  },
  taxes: {
    issPis: 0.0565,               // alíquota ISS + PIS/COFINS (varia por município)
    ecad: 0.04,                   // alíquota ECAD (varia por tipo de evento)
  },
}
```

### Show (estado mutável — criado a partir de um Venue)
```js
{
  id: 'abc123',                   // uid() aleatório
  venueId: 'espaço-unimed-sp',    // referência ao venue template
  date: '2026-08-15',
  cachet: 150000,                 // cachê do artista em R$

  // Distribuição de ingressos (aplicada em lote)
  splits: { meia: 0.4, solidario: 0.5, inteira: 0.1 },

  // Preço do solidário como % do preço inteira (padrão: 67.5)
  solidarioPct: 67.5,

  // Tipo de locação para este show (sobrescreve o template do venue)
  locacaoOverride: {
    type: 'template',             // 'template' | 'fixed' | 'pct'
    value: 0,                     // valor em R$ (quando type === 'fixed')
    pct: 0,                       // percentual (quando type === 'pct')
  },

  // Custos adicionais por categoria (linhas livres)
  categoryCosts: {
    venue:    [{ id, description, value }],
    prod:     [{ id, description, value }],
    tecnica:  [{ id, description, value }],
    logist:   [{ id, description, value }],
    outros:   [{ id, description, value }],
  },

  // Overrides de custos do template (null = usa o template do venue)
  costOverrides: {
    producao: null,               // null = usa venue.costs.producao
    logistica: 30000,             // valor sobrescrito
    backline: null,
    iluminacao: null,
    som: null,
    led: null,
    outros: null,
  },

  // Setores (cópia do venue, mutável por show)
  sectors: [
    {
      id: 'platinum',
      name: 'PLATINUM',
      capacity: 360,
      qtyMeia: 144,               // Math.round(capacity * splits.meia)
      qtySolidario: 180,          // Math.round(capacity * splits.solidario)
      qtyInteira: 36,             // Math.round(capacity * splits.inteira)
      priceMeia: 250,             // priceInteira * 0.5
      priceSolidario: 337.5,      // priceInteira * (solidarioPct / 100)
      priceInteira: 500,
    }
  ],

  // Campo legado — mantido para compatibilidade
  extraCosts: [],
}
```

### Budget (orçamento)
```js
{
  id: 'xyz789',
  name: 'Turnê Brasil 2026',
  artist: 'Tim Bernardes',
  createdAt: '2026-01-15T10:00:00.000Z',
  updatedAt: '2026-02-01T14:30:00.000Z',
  shows: [ /* array de Show */ ],
}
```

---

## Engine de Cálculo

### `calcSectorRevenue(sector)`
Receita de um setor: `(qtyMeia × priceMeia) + (qtySolidario × priceSolidario) + (qtyInteira × priceInteira)`

### `calcShow(show, venueMap)` → objeto com todos os KPIs
```
revenue       = soma calcSectorRevenue de todos os setores
ecad          = revenue × venue.taxes.ecad
issPis        = revenue × venue.taxes.issPis

locacao       = depende de show.locacaoOverride.type:
                  'template' → usa venue.costs.locacaoValue (fixo ou % da receita)
                  'fixed'    → show.locacaoOverride.value
                  'pct'      → show.locacaoOverride.pct / 100 × revenue

producao      = show.costOverrides.producao ?? venue.costs.producao
(idem para logistica, backline, iluminacao, som, led, outros)

catCosts      = soma das linhas em show.categoryCosts por categoria
catExtrasTotal = soma de todos os catCosts

cachet        = show.cachet

totalCosts    = ecad + issPis + locacao + producao + logistica +
                backline + iluminacao + som + led + outros +
                extras + catExtrasTotal + cachet

result        = revenue - totalCosts
margin        = result / revenue
totalSold     = soma das qtys de todos os setores
occupancy     = totalSold / totalCapacity
ticketMedio   = revenue / totalSold
```

### `calcBudget(budget, venueMap)` → KPIs consolidados
Agrega os resultados de `calcShow()` para todos os shows do orçamento.

---

## Deploy no Vercel

O projeto é um site estático — o deploy é simples:

### Método 1: Upload direto (manual)
1. Acesse [vercel.com](https://vercel.com) e faça login
2. Clique em **Add New → Project**
3. Escolha **"Import Third-Party Git Repository"** ou use **deploy via CLI**
4. Alternativamente, arraste a pasta do projeto para [vercel.com/new](https://vercel.com/new)

### Método 2: Via GitHub (recomendado para atualizações contínuas)
1. O repositório já existe em `github.com/gabrieljandrade/venue-manager`
2. No Vercel: **Add New → Project → Import Git Repository**
3. Conecte o GitHub e selecione `venue-manager`
4. Em **Root Directory**, defina como `venue-manager` (onde está o `index.html`)
5. **Framework Preset**: Other (sem build)
6. Clique em **Deploy**
7. A partir daí, qualquer push para `main` faz redeploy automático

### `vercel.json` (já incluído no projeto)
```json
{
  "rewrites": [{ "source": "/(.*)", "destination": "/index.html" }]
}
```
Garante que qualquer URL sirva o `index.html` (útil caso o Vercel tente resolver rotas).

---

## Como Adicionar um Novo Venue

Edite o array `VENUES` no início do `index.html`, seguindo exatamente a estrutura dos venues existentes:

```js
{
  id: 'meu-venue-cidade',         // kebab-case, único
  name: 'Nome do Venue',
  city: 'Cidade',
  state: 'UF',
  totalCapacity: 2000,
  cortesias: 100,
  sectors: [
    {
      id: 'pista',
      name: 'PISTA',
      capacity: 1500,
      priceInteira: 200,
      splitMeia: 0.4,
      splitSolidario: 0.5,
      splitInteira: 0.1,
    },
    // adicione mais setores conforme necessário
  ],
  costs: {
    locacaoType: 'fixed',         // 'fixed' ou 'pct'
    locacaoValue: 80000,          // R$ se fixed, decimal (ex: 0.15) se pct
    producao: 150000,
    logistica: 26400,
    backline: 5000,
    iluminacao: 30000,
    som: 30000,
    led: 15000,
    outros: 800,
  },
  taxes: {
    issPis: 0.0565,               // verifique a alíquota municipal
    ecad: 0.05,
  },
}
```

---

## Como Expandir o Projeto

### Adicionar uma nova categoria de custo
1. Adicione a chave em `COST_LABELS` (label de exibição)
2. Adicione-a na categoria relevante em `DRE_CATS` (para exibição na DRE)
3. Adicione-a em `OVERRIDE_CATS` (se for editável por show)
4. Inclua o cálculo em `calcShow()` (seguindo o padrão `ov.chave ?? venue.costs.chave`)
5. Adicione o campo em `venue.costs` nos venues existentes

### Adicionar um novo campo ao Show
1. Adicione o campo com valor default em `makeShow()`
2. Se for calculado, adicione em `calcShow()`
3. Adicione o UI em `ShowForm` para edição
4. Se relevante para a DRE, adicione em `DREPanel` e `DREConsolidated`

### Migrar para um projeto com build (Vite + React)
Quando o projeto crescer e precisar de performance melhor:
1. `npm create vite@latest venue-manager -- --template react`
2. Mover o conteúdo do `<script type="text/babel">` para componentes `.jsx` separados
3. Trocar CDN imports por `npm install react react-dom xlsx`
4. Manter a mesma lógica — a engine de cálculo é pura e facilmente portável

---

## Decisões Técnicas e Trade-offs

| Decisão | Por quê | Limitação |
|---------|---------|-----------|
| Single HTML file | Zero config, deploy imediato | ~2-3s de carregamento (Babel em runtime) |
| localStorage | Sem backend necessário | Dados locais por máquina, sem colaboração |
| React Context (VenueCtx) | Evitar prop-drilling do venueMap | OK para essa escala — Redux seria overkill |
| Babel Standalone no browser | JSX sem build step | Não recomendado para produção em larga escala |
| Tailwind CDN | Sem build, classes dinâmicas ok | Bundle maior do que o necessário |
| Venues como dados estáticos no código | Simples de manter | Precisar editar o código para adicionar venues |

---

## Variáveis de Negócio Relevantes

| Termo | Descrição |
|-------|-----------|
| **Split** | Proporção de ingressos por tipo (meia/solidário/inteira) |
| **Solidário** | Ingresso com desconto para doação de alimento — preço configurável como % da inteira |
| **Locação** | Custo de aluguel do venue — pode ser valor fixo ou % da receita bruta |
| **ECAD** | Escritório Central de Arrecadação e Distribuição — taxa sobre receita de eventos musicais |
| **ISS+PIS/COFINS** | Impostos municipais e federais — alíquota varia por cidade e regime tributário |
| **Ticket Médio** | Receita total / Total de ingressos vendidos |
| **Ocupação** | Total vendido / Capacidade total do venue |
| **DRE** | Demonstração do Resultado do Exercício — demonstrativo financeiro |
| **Cachê** | Pagamento ao artista — custo fixo por show |

---

## Estrutura de Arquivos do Projeto

```
venue-manager/
├── index.html       → Toda a aplicação (HTML + CSS + JS/JSX)
├── vercel.json      → Configuração de deploy estático no Vercel
└── README.md        → Este arquivo
```

---

*Projeto desenvolvido para gestão interna de projeções financeiras de turnês. Dados de venues e custos baseados em contratos reais — atualizar conforme necessário.*
