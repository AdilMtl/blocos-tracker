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

## Fase 2 (futura — não nesta sprint)
- FAB (+): bottom sheet com opções (Alimento / Treino / Medição)
- Gráfico semanal de kcal na Home
- Auto-check de hábitos ao salvar dados
