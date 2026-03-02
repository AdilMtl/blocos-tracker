# Kcal.ix — Plano de Desenvolvimento

## Histórico de fases concluídas

| Fase | Descrição | Versão |
|---|---|---|
| Fase 1 (Etapas 1–8) | Rename + reestruturação de UX (nav, views, Home) | v1.5.0 |
| Fase 2 | Auto-check de hábitos, remoção do FAB | v1.6.0 |
| Fase 3 | Energy Analytics Dashboard (BMR, gráfico semanal, projeção) | v1.7.0 |
| Fase 4 | Timer de pausa no treino + cronômetro + notificação nativa | v1.8.0 |
| Fase 5 | Banner PWA (iOS + Android), diet auto-check | v1.8.3 |
| Fase 6 | Accordion alimentos no Diário: sub-accordions, modal de porção, UX de refeições | v1.9.0 |

---

## Regras durante a implementação

1. Nunca renomear storage keys — dados reais do usuário
2. Testar cada etapa antes de avançar
3. Commits pequenos por etapa
4. Marcar [x] neste arquivo ao concluir cada item

---

## Fase 7 — Food Panel Drawer (spec aprovada em: 2026-03-01)

### Visão geral

Redesenho da UX de adição de alimentos no Diário. Substitui o sistema de
sub-accordions aninhados por um **bottom drawer fixo** inspirado no MyFitnessPal,
com layout dividido em zona de busca (70%) e zona de log do dia (30%).

### Motivação

- Lista de alimentos cresce indefinidamente, sem altura máxima
- "Adicionados hoje" e "Alimento personalizado" ficam soterrados abaixo da lista
- Sem botão ✕ para limpar o campo de busca
- Sem aba de "Recentes" para acessar alimentos usados frequentemente

### Layout do drawer

```
┌────────────────────────────────────────┐  ← slide up desde o bottom
│  🍽️ Adicionar alimentos           ✕   │  ← header fixo
├────────────────────────────────────────┤
│ [🔍 Buscar alimento...      ] [ ✕ ]   │  ← search + clear button
│ [Recentes][Todos][Cat 1][Cat 2] →      │  ← cat tabs (scroll horizontal)
│                                        │
│  food-item                             │  ← grid scrollável (flex:1)
│  food-item                             │
│  food-item                             │
│  ...                                   │
│                                        │
│ ┌────────────────────────────────────┐ │
│ │ ➕ Criar alimento personalizado    │ │  ← sticky, sempre visível
│ └────────────────────────────────────┘ │
├────────────────────────────────────────┤
│ 📋 Adicionados hoje · N itens      ▾  │  ← peek 44px, tap p/ expandir
│   (lista dos itens ao expandir)        │  ← max ~33vh
└────────────────────────────────────────┘
```

### Detalhes de comportamento

**Drawer:**
- Altura total: `88dvh` com `safe-area-inset-bottom`
- Overlay escuro atrás, tap fecha
- `#accAlimentos` no Diário vira apenas o botão trigger (não mais um accordion)

**Tab "Recentes":**
- Aparece antes de "Todos" somente se existir histórico no `foodLog`
- Lista até 10 alimentos únicos por `foodId`, ordenados por `at` timestamp desc
- Buscados em todos os dias do `foodLog.days`

**Botão ✕ no search:**
- Visível somente quando `foodSearchTerm !== ""`
- Limpa o input e re-renderiza o grid

**"Alimento personalizado" (sticky):**
- Sempre visível como última linha antes do peek
- Ao clicar, abre mini-modal (overlay separado) com o form atual dos 6 campos

**"Adicionados hoje" (peek):**
- Colapsado por padrão: 44px, mostra count
- Toque expande até `max-height: 33vh`, com overflow-y scroll
- Ao remover item, re-renderiza contagem e lista sem fechar o drawer

---

### ETAPA 1 — CSS do drawer

- [ ] Remover regra `#accAlimentos.open > .acc-content { max-height:9000px }`
- [ ] `.food-drawer-overlay`: fixed, fullscreen, z-index 310, backdrop semitransparente
- [ ] `.food-drawer`: fixed, bottom 0, width 100%, height 88dvh, z-index 311, slide-up animation, flex column
- [ ] `.fd-header`: flex, shrink 0, título + botão ✕
- [ ] `.fd-search-wrap`: position relative, shrink 0; `.fd-search-clear`: botão ✕ absoluto à direita
- [ ] `.fd-cat-tabs`: igual ao `.cat-tabs` atual (reutilizar ou renomear)
- [ ] `.fd-grid`: flex 1, overflow-y auto, `-webkit-overflow-scrolling: touch`
- [ ] `.fd-custom-row`: shrink 0, botão full-width, borda topo
- [ ] `.fd-peek`: shrink 0, height 44px, cursor pointer, borda topo, transição max-height
- [ ] `.fd-peek.open`: max-height 33vh, overflow-y auto
- [ ] `.fd-peek-header`: flex, align-center, padding, chevron rotacionável

---

### ETAPA 2 — HTML

- [ ] `#accAlimentos`: manter o `<div class="accordion">` mas remover o conteúdo dos sub-accordions; trigger chama `openFoodDrawer()` em vez de `toggleAcc()`
- [ ] Após `</div><!-- /app -->`: adicionar `#foodDrawerOverlay` + `#foodDrawer`
- [ ] Estrutura do drawer: `#fdHeader` / `#fdSearchWrap` (input `#fdSearch` + btn `#fdClear`) / `#fdCatTabs` / `#fdGrid` / `#fdCustomRow` / `#fdPeek` (`#fdPeekHeader` + `#fdLog`)
- [ ] Remover os 3 divs `#accFoodSearch`, `#accFoodLog`, `#accCustomFood` do viewDiario

---

### ETAPA 3 — JS: helpers e funções de render

- [ ] `getRecentFoods()`: varre `foodLog.days`, ordena entries por `at` desc, deduplica por `foodId`, retorna array de food objects (até 10)
- [ ] `openFoodDrawer()`: adiciona class `.open` no overlay + drawer; chama `renderFoodDrawer()`
- [ ] `closeFoodDrawer()`: remove class `.open`
- [ ] `renderFoodDrawer()`: chama `renderFdCatTabs()` + `renderFdGrid()` + `renderFdLog()`
- [ ] `renderFdCatTabs()`: baseada em `renderCatTabs()`, mas targets `#fdCatTabs`; adiciona tab "Recentes" antes de "Todos" se `getRecentFoods().length > 0`; quando "Recentes" ativo, passa lista direto para `renderFdGrid()`
- [ ] `renderFdGrid()`: baseada em `renderFoodGrid()`, mas target `#fdGrid`; usa `foodSearchTerm` da mesma var global
- [ ] `renderFdLog()`: baseada em `renderFoodLog()`, mas target `#fdLog` e `#fdPeekHeader` (atualiza count)

---

### ETAPA 4 — JS: event listeners

- [ ] `#foodDrawerOverlay` click → `closeFoodDrawer()`
- [ ] `#fdHeader` botão ✕ click → `closeFoodDrawer()`
- [ ] `#fdSearch` input → atualiza `foodSearchTerm`, mostra/esconde `#fdClear`, chama `renderFdGrid()`
- [ ] `#fdClear` click → limpa `#fdSearch`, zera `foodSearchTerm`, chama `renderFdGrid()`
- [ ] `#fdPeekHeader` click → toggle class `.open` em `#fdPeek`, rotaciona chevron
- [ ] `#fdCustomRow` click → abre mini-modal `#customFoodModal` (reusa form existente ou abre accordion inline)
- [ ] Adaptar `addFoodToMeal()` e `removeFoodLogEntry()` para chamar `renderFdLog()` além das renders existentes
- [ ] Adaptar `openTab("diario")` para não mais chamar `renderFoodPanel()` (só chama quando drawer abre)

---

### Critérios de feito (Fase 7)

- [ ] Drawer abre/fecha com animação suave, overlay fecha ao clicar fora
- [ ] Search com ✕ funciona corretamente em mobile (teclado não quebra layout)
- [ ] Tab "Recentes" aparece somente quando há histórico, lista até 10 itens únicos
- [ ] Food grid limitado à área do drawer, sem scroll da página
- [ ] "Alimento personalizado" sempre visível ao rolar o grid
- [ ] Peek "Adicionados hoje" expande/colapsa sem fechar o drawer
- [ ] Funciona em 375px sem overflow horizontal
- [ ] Nenhuma storage key alterada
- [ ] `#foodModal` (modal de porção) continua funcionando sem mudanças
