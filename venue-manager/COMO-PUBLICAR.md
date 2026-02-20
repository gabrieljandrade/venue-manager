# Como publicar o Venue Manager para sua equipe

## Opção 1 — Vercel (recomendado, gratuito, 5 minutos)

1. Crie uma conta gratuita em **vercel.com**
2. Clique em **"Add New Project"**
3. Clique em **"Import Git Repository"** — ou use o **Vercel CLI** (abaixo)
4. Faça upload do arquivo `index.html` como um repositório GitHub (ou arraste a pasta inteira)
5. Pronto — você recebe uma URL do tipo `venue-manager-xxxx.vercel.app`

**Via Vercel CLI (mais simples):**
```bash
npm install -g vercel
vercel login
cd venue-manager/
vercel --yes
```

## Opção 2 — Netlify Drop (mais fácil ainda)

1. Acesse **app.netlify.com/drop**
2. Arraste a pasta `venue-manager/` para a página
3. Pronto — URL instantânea como `random-name.netlify.app`

## Opção 3 — Abrir localmente (sem internet)

Abra o arquivo `index.html` diretamente no Chrome/Firefox.
Funciona offline, salva os dados no navegador.

---

## Funcionalidades do app

- **12 venues pré-configurados** com todos os custos e taxas
- **Cálculo automático** de DRE por show em tempo real
- **Múltiplos shows** por orçamento (turnê completa)
- **Precificação por setor** (inteira, meia entrada, solidário, etc.)
- **Custos editáveis** — substitua qualquer valor padrão do venue
- **Custos extras** — adicione itens pontuais por show
- **DRE consolidada** com todos os shows lado a lado
- **Exportação para Excel** com aba por show
- **Impressão / PDF** via Ctrl+P
- **Histórico salvo** no navegador (localStorage)

## Venues incluídos

| Venue | Cidade | Capacidade |
|-------|--------|-----------|
| Espaço Unimed | São Paulo, SP | 3.250 |
| Vivo Rio | Rio de Janeiro, RJ | 2.066 |
| BeFly Hall | Belo Horizonte, MG | 3.574 |
| Araújo Viana | Porto Alegre, RS | 4.228 |
| Teatro Guaíra | Curitiba, PR | 2.119 |
| Ópera de Arame | Curitiba, PR | 1.644 |
| Teatro Amazonas | Manaus, AM | 701 |
| Cine Teatro São Luiz | Fortaleza, CE | 1.042 |
| Teatro da Paz | Belém, PA | 900 |
| Teatro Castro Alves | Salvador, BA | 1.542 |
| Arena Jockey | Rio de Janeiro, RJ | 3.550 |
| Espaço Ibirapuera | São Paulo, SP | 15.800 |

---

> Para adicionar novos venues, edite o array `VENUES` no arquivo `index.html`
> na seção marcada como `// ─── VENUE TEMPLATES ───`.
