# Kcal.ix — Plano de Reestruturação de UX

Spec aprovada em: 2026-02-28
Status geral: ✅ Concluído — Todas as etapas implementadas

---

## Passos de implementação

### ETAPA 1 — Rename: Blocos Tracker → Kcal.ix
- [x] `<title>` no index.html
- [x] `<h1>` no header do index.html
- [x] `AI_SYSTEM_PROMPT` (menção ao app)
- [x] Nome do arquivo de export JSON (download)
- [x] `manifest.json`: name + short_name + description
- [x] `sw.js`: CACHE_NAME bump (blocos-v5 → kcalix-v1)
- **Nota:** storage keys (`blocos_tracker_*`) NÃO mudam — contêm dados reais do usuário

---

### ETAPA 2 — Bottom-nav: 6 → 5 abas ✅
- [x] Renomear IDs: `viewTracker` → `viewDiario`, ajustar referências no JS
- [x] viewSettings + viewCalc → merged em `viewMais`
- [x] `viewMeasure` → `viewCorpo`
- [x] Atualizar HTML da bottom-nav (5 itens: Home | Diário | Treino | Corpo | Mais)
- [x] Atualizar `openTab()` no JS com novos IDs/labels
- [x] Criar `<div id="viewHome">` como primeira view (stub renderHomeDashboard)
- [x] viewAlimentos mantido no HTML, removido da nav (Etapa 3 move conteúdo)

---

### ETAPA 3 — Alimentos dentro do Diário ✅
- [x] Mover HTML de `#viewAlimentos` para accordion no fim de `#viewDiario`
- [x] `openTab("diario")` chama `renderFoodPanel()`
- [x] Accordion trigger também chama `renderFoodPanel()` ao abrir
- [x] `<div id="viewAlimentos">` removido do HTML

---

### ETAPA 4 — Habit Tracker move para Home ✅
- [x] `#habitTracker` criado dentro de `#viewHome`
- [x] Removido do topo de `#viewDiario`
- [x] `renderHabitTracker()` usa `$('#habitTracker')` — funciona sem mudança

---

### ETAPA 5 — CSS da Home ✅
- [x] `.home-grid`, `.home-action-card`, `.home-action-icon`, `.home-action-label`
- [x] `.home-kcal-bar`, `.home-kcal-fill`, `.home-macro-bar`, `.home-macro-fill`
- [x] `.home-greeting`, `.home-date-sub`, `.home-kcal-row`, `.home-macro-row`
- [x] `.fab`: fixed, z-index 200, cor accent, active scale

---

### ETAPA 6 — HTML da Home ✅
- [x] Saudação contextual + data formatada (`#homeGreeting`, `#homeDate`)
- [x] `#habitTracker` no topo
- [x] Card "Hoje" clicável: barra kcal + 3 mini-barras P/C/G
- [x] Grid 2×2: Diário / Treino / Corpo / IA Export
- [x] FAB `#fabAdd` (placeholder — Fase 2 implementa bottom sheet)

---

### ETAPA 7 — JS da Home ✅
- [x] `renderHomeDashboard()`: saudação, data, kcal, P/C/G em gramas vs meta
- [x] Barra kcal usa `settings.kcalPerBlock` × blocos consumidos
- [x] Macro bars usam gramas: `totals() × settings.blocks.*` vs `settings.goals.*`
- [x] Bloco IA Export → `document.getElementById('btnExportAI').click()`

---

### ETAPA 8 — Comportamento de abertura (sessionStorage) ✅
- [x] `openTab()` faz `sessionStorage.setItem('kcalix_tab', which)` a cada troca
- [x] No boot: lê sessionStorage → abre aba salva ou Home se não existir
- [x] sessionStorage é per-session: fecha aba → reabre → volta para Home ✓
- [x] Troca de app no celular: sessionStorage preservada → mantém aba ✓

---

## Regras durante a implementação

1. Nunca renomear storage keys — dados reais do usuário
2. Testar cada etapa antes de avançar
3. Commits pequenos por etapa
4. Marcar [x] neste arquivo ao concluir cada item

---

## Fase 2 ✅ Concluído (2026-02-28 — v1.6.0)
- [x] FAB (+) removido — redundante com grid da Home
- [x] Auto-check de hábitos ao salvar dados (log, dieta, treino, cardio, medidas)
- [ ] Gráfico semanal de kcal → movido para Fase 3

---

## Fase 3 — Energy Analytics Dashboard (spec aprovada em: 2026-02-28)

### Visão geral
Painel de análise energética diária e semanal na Home. Combina calorias
consumidas, gasto basal (TDEE), gasto de treino e exibe saldo com
projeção de evolução corporal (ganho de massa / perda de gordura).

### Dados disponíveis (sem nova infra de storage)

| Variável           | Fonte                                         |
|--------------------|-----------------------------------------------|
| kcalConsumido(dia) | data.days[date].meals + settings.kcalPerBlock |
| kcalTreino(dia)    | treinoData.days[date].kcal (já salvo)         |
| BMR                | calc (CALC_KEY) → bmrKatch ou bmrMifflin      |
| TDEE               | calc.activity × BMR                           |
| kcalGasto(dia)     | TDEE + kcalTreino(dia)                        |
| Saldo(dia)         | kcalConsumido - kcalGasto                     |
| Meta kcal          | settings.goals.kcal                           |

**Nota:** BMR é calculado uma vez a partir do `calc` salvo — não varia
por dia. Para dias sem treino registrado, kcalTreino = 0.

---

### ETAPA 1 — Helper: getEnergyForDate(date) ✅
Função pura que retorna { consumed, bmr, tdee, exercise, total, balance }

```
- [x] Definir getEnergyForDate(date) no escopo do IIFE
- [x] consumed: soma meals do data.days[date] × kcalPerBlock
- [x] bmr: recalcular de calc (Katch se leanKg disponível, senão Mifflin)
      → se calc vazio (nenhum dado inserido), bmr = null
- [x] tdee: bmr × (calc.activity || 1.55)
- [x] exercise: treinoData.days[date]?.kcal || 0
- [x] total: tdee + exercise
- [x] balance: consumed - total (null se consumed=0 ou tdee=null)
- [x] Retorna null para cada campo que não pode ser calculado
```

---

### ETAPA 2 — KPIs de energia (card "Hoje") ✅

- [x] HTML: div#energyCard no viewHome (abaixo do card macros)
- [x] 4 KPIs inline: consumido / gasto / treino / saldo
- [x] Saldo com cor semântica: verde (<0 déficit) / accent2 (>0 superávit)
- [x] Barra de progresso: consumido vs meta
- [x] Se JP7 não salvo: exibe só consumido + msg "Configure JP7"
- [x] CSS: .energy-kpi-row, .energy-kpi, .energy-kpi-val, .energy-bar-wrap

---

### ETAPA 3 — Gráfico semanal de barras ✅

- [x] HTML: div#weekEnergyChart vazio no viewHome
- [x] JS: renderWeekEnergyChart() — 7 dias da semana atual (Seg→Dom)
- [x] Barras duplas sobrepostas: gasto (surface3, full) + consumido (accent, 55%)
- [x] Linha tracejada = settings.goals.kcal
- [x] Dia atual: borda accent2
- [x] Dias sem dados: opacidade reduzida
- [x] Dias futuros: sem barra
- [x] CSS: .week-chart-wrap, .week-bars, .week-bar-group, .week-meta-line

---

### ETAPA 4 — Projeção semanal ✅

- [x] Média de balance dos dias com dados (só dias não-futuros com balance!=null)
- [x] Converter para kg: média × 7 / 7700
- [x] Exibe se ≥ 3 dias com dados (📉 déficit / 📈 superávit)
- [x] CSS: .energy-projection

---

### ETAPA 5 — Integração com renderHomeDashboard() ✅

- [x] Chamar renderEnergyCard() dentro de renderHomeDashboard()
- [x] Chamar renderWeekEnergyChart() dentro de renderHomeDashboard()
- [x] Atualiza toda vez que Home é aberta (via openTab)

---

### Limitações documentadas (v1 da feature)

1. BMR fixo: usa último calc salvo, não calcula por peso do dia
2. TDEE = BMR × fator geral (não ajusta por tipo de dia — treino vs descanso)
3. kcalTreino é estimativa (fórmula interna, não dispositivo externo)
4. foodLog (alimentos detalhados) não entra no cálculo — só blocos do Diário
5. Dias sem refeição registrada são excluídos da projeção

---

### Critérios de feito da Fase 3 ✅ (implementado 2026-02-28)

- [x] Card energia visível na Home com 4 KPIs
- [x] Barra consumido vs meta renderizada
- [x] Gráfico 7 dias com barras duplas e linha meta
- [x] Projeção exibida se ≥ 3 dias com dados
- [x] Sem BMR configurado: card degradado mas sem erro
- [x] Funciona em 375px sem overflow horizontal
- [x] Re-render ao abrir Home
