# Spec v2.8.0 — Analytics de Volume por Grupo Muscular

**Status**: CONCLUÍDO — deployado em v2.8.0 (2026-03-05, commit e1577b5)
**Data**: 2026-03-05
**Fonte técnica**: Protocolos Lucas Campos (RP-based) + NotebookLM
**Motivação**: O modal de historico (#tmplHistModal) acompanha sessoes e progressao por exercicio, mas nao oferece visibilidade da distribuicao de volume entre grupos musculares ao longo da semana.

---

## PLANO DE IMPLEMENTACAO (4 partes — executar em ordem)

Cada parte e independentemente testavel antes de avancar para a proxima.
Nenhuma parte quebra funcionalidade existente se implementada corretamente.

---

## Parte 0 — Custom exercises: campo de grupos secundarios

### Por que comecar aqui
Sem isso, exercicios custom no analytics so contariam o grupo primario. Melhor resolver antes de construir o analytics.

### Mudancas no #customExModal (criacao)

Apos o `<select id="cexGrupo">`, adicionar:

```html
<div class="form-row" style="margin-bottom:20px;" id="cexSecRow">
  <label>Grupos secundarios <span style="color:var(--text3);font-weight:400;">(opcional)</span></label>
  <div id="cexSecChips" style="display:flex;flex-wrap:wrap;gap:6px;margin-top:6px;"></div>
</div>
```

CSS dos chips:
```css
.sec-chip {
  padding: 4px 10px;
  border-radius: 20px;
  border: 1px solid var(--line);
  font-size: 12px;
  font-weight: 600;
  color: var(--text2);
  background: transparent;
  cursor: pointer;
  transition: background 0.15s, color 0.15s;
}
.sec-chip.active {
  background: var(--accent);
  color: #fff;
  border-color: var(--accent);
}
```

### Funcao renderSecChips(excludeGrupo)

Chamada sempre que o select #cexGrupo mudar, e na abertura do modal:

```js
const renderSecChips = (excludeGrupo) => {
  const wrap = $("#cexSecChips");
  wrap.innerHTML = "";
  Object.keys(EXERCISE_DB).forEach(g => {
    if (g === excludeGrupo) return;
    const btn = document.createElement("button");
    btn.type = "button";
    btn.className = "sec-chip";
    btn.textContent = g;
    btn.dataset.grupo = g;
    btn.addEventListener("click", () => btn.classList.toggle("active"));
    wrap.appendChild(btn);
  });
};
```

Listener no select:
```js
$("#cexGrupo").addEventListener("change", () => renderSecChips($("#cexGrupo").value));
```

### Leitura dos chips ao salvar (saveCustomEx)

```js
const secundarios = [...$("#cexSecChips").querySelectorAll(".sec-chip.active")]
  .map(b => b.dataset.grupo);
// Adicionar ao objeto salvo: { id, nome, grupo, secundarios }
```

### Schema custom exercise (retrocompativel)

```js
// ANTES:
{ id, nome, grupo }

// DEPOIS:
{ id, nome, grupo, secundarios: [] }
// Exercicios existentes sem secundarios: tratados como [] por padrao — nenhum dado perdido
```

### Mudanca em openExSelCustomRename()

Adicionar os mesmos chips de toggle apos o input de nome e select de grupo (se houver).
Pre-marcar os chips com base em `ex.secundarios` existentes.

### Helper getSecondary(exInfo)

Adicionar apos a definicao de `exById()`:

```js
const getSecondary = (exInfo) => {
  if (!exInfo) return [];
  // Custom: usa campo secundarios do proprio objeto
  if (customExercises.some(e => e.id === exInfo.id))
    return exInfo.secundarios || [];
  // Built-in: usa lookup estatico
  return EX_SECONDARY[exInfo.id] || [];
};
```

---

## Parte 1 — Constantes: EX_SECONDARY e MUSCLE_LANDMARKS

Adicionar antes de `const CARDIO_TYPES` (apos EXERCISE_DB).

### EX_SECONDARY

Mapeia IDs de exercicios built-in para grupos secundarios.
Regra: motor primario = 1.0 serie, sinergista = 0.5 serie.

```js
const EX_SECONDARY = {
  // Peito (primario) — recruta Triceps e Ombros
  supino_reto:           ["💪 Tríceps", "💪 Ombros"],
  supino_inclinado:      ["💪 Tríceps", "💪 Ombros"],
  supino_halter:         ["💪 Tríceps", "💪 Ombros"],
  supino_incl_halter:    ["💪 Tríceps", "💪 Ombros"],
  crucifixo:             ["💪 Ombros"],
  crossover:             [],
  peck_deck:             [],
  supino_maquina:        ["💪 Tríceps"],
  flexao:                ["💪 Tríceps", "💪 Ombros"],
  // Costas (primario) — recruta Biceps
  puxada_frontal:        ["💪 Bíceps"],
  puxada_triang:         ["💪 Bíceps"],
  remada_curvada:        ["💪 Bíceps"],
  remada_halter:         ["💪 Bíceps"],
  remada_cavalinho:      ["💪 Bíceps"],
  remada_baixa:          ["💪 Bíceps"],
  pulldown:              ["💪 Bíceps"],
  barra_fixa:            ["💪 Bíceps"],
  remada_maquina:        ["💪 Bíceps"],
  // Pernas — quad (primario)
  agachamento_livre:     ["🍑 Glúteos", "🧱 Core"],
  agachamento_smith:     ["🍑 Glúteos"],
  leg_press:             ["🍑 Glúteos"],
  leg_press_horiz:       ["🍑 Glúteos"],
  cadeira_extensora:     [],
  hack_squat:            ["🍑 Glúteos"],
  passada:               ["🍑 Glúteos"],
  bulgaro:               ["🍑 Glúteos"],
  // Pernas — posterior/isquio (primario)
  cadeira_flexora:       [],
  mesa_flexora:          [],
  stiff:                 ["🍑 Glúteos"],
  // Gluteos (primario) — frequentemente co-primarios com posterior
  hip_thrust_barra:      ["🦵 Pernas"],
  hip_thrust_maquina:    ["🦵 Pernas"],
  gluteo_maquina:        [],
  gluteo_cabo:           [],
  elevacao_pelvica:      ["🦵 Pernas"],
  stiff_romeno:          ["🦵 Pernas"],
  agachamento_sumo:      ["🦵 Pernas"],
  // Ombros (primario)
  desenv_halter:         ["💪 Tríceps"],
  desenv_barra:          ["💪 Tríceps"],
  desenv_maquina:        ["💪 Tríceps"],
  elevacao_lateral:      [],
  elevacao_lateral_cabo: [],
  elevacao_frontal:      ["🏋️ Peito"],
  face_pull:             ["💪 Bíceps"],
  encolhimento:          [],
  crucifixo_inverso:     [],
  // Panturrilha
  panturrilha_pe:        [],
  panturrilha_sentado:   [],
  // Isolados — sem secundarios significativos
  abdutora:              ["🍑 Glúteos"],
  adutora:               [],
  abdominal_crunch:      [],
  abdominal_infra:       [],
  prancha:               [],
  prancha_lateral:       [],
  abdominal_maquina:     [],
  rotacao_russa:         [],
  roda_abdominal:        [],
};
```

### MUSCLE_LANDMARKS

Baseado em Lucas Campos. Pernas separadas em Quad e Posterior para analytics.

```js
const MUSCLE_LANDMARKS = {
  "🏋️ Peito":     { mev: 10, mav: 15, mrv: 25 },
  "🦅 Costas":    { mev: 10, mav: 15, mrv: 25 },
  "🦵 Quad":      { mev:  8, mav: 15, mrv: 25 },
  "🦵 Posterior": { mev:  6, mav: 12, mrv: 20 },
  "🍑 Glúteos":   { mev: 15, mav: 20, mrv: 30 },
  "💪 Ombros":    { mev:  6, mav: 12, mrv: 20 },
  "💪 Bíceps":    { mev:  6, mav: 12, mrv: 20 },
  "💪 Tríceps":   { mev:  6, mav: 12, mrv: 20 },
  "🧱 Core":      { mev:  4, mav: 10, mrv: 16 },
};

// Ordem de exibicao dos cards
const MUSCLE_ORDER = [
  "🏋️ Peito","🦅 Costas","🦵 Quad","🦵 Posterior",
  "🍑 Glúteos","💪 Ombros","💪 Bíceps","💪 Tríceps","🧱 Core"
];

// Mapeamento: exercicio de "Pernas" -> qual analytics group
const QUAD_IDS = [
  "agachamento_livre","agachamento_smith","leg_press","leg_press_horiz",
  "cadeira_extensora","hack_squat","passada","bulgaro","abdutora","adutora"
];
const POST_IDS = ["cadeira_flexora","mesa_flexora","stiff","stiff_romeno"];
// Exercicios "Pernas" nao listados -> Quad por padrao
// Custom com grupo "Pernas" -> Quad por padrao
```

---

## Parte 2 — Funcao calcMuscleVolume(startDate, endDate)

Adicionar apos `getAllTmplSessions`.

```js
// startDate e endDate: strings "YYYY-MM-DD"
const calcMuscleVolume = (startDate, endDate) => {
  const result = {};
  MUSCLE_ORDER.forEach(g => { result[g] = { direct: 0, indirect: 0, total: 0 }; });

  const resolvePrimaryGroup = (exInfo) => {
    if (!exInfo) return null;
    const g = exInfo.grupo;
    if (g === "🦵 Pernas") {
      if (POST_IDS.includes(exInfo.id)) return "🦵 Posterior";
      return "🦵 Quad";
    }
    // Gluteos, Peito, Costas, Ombros, Biceps, Triceps, Core -> direto
    return MUSCLE_ORDER.find(m => m === g) || null;
  };

  const days = treinoData.days || {};
  for (const [date, day] of Object.entries(days)) {
    if (date < startDate || date > endDate) continue;
    for (const ex of (day.exercicios || [])) {
      const exInfo = exById(ex.exercicioId);
      const primaryGroup = resolvePrimaryGroup(exInfo);
      const secondaries = getSecondary(exInfo);
      const validSets = (ex.series || []).filter(s => (Number(s.reps) || 0) > 0).length;
      if (validSets === 0) continue;
      if (primaryGroup && result[primaryGroup]) {
        result[primaryGroup].direct += validSets;
        result[primaryGroup].total  += validSets;
      }
      for (const sec of secondaries) {
        const secGroup = MUSCLE_ORDER.find(m => m === sec);
        if (secGroup && result[secGroup]) {
          result[secGroup].indirect += validSets * 0.5;
          result[secGroup].total    += validSets * 0.5;
        }
      }
    }
  }
  return result;
};
```

### calcMuscleAvg4weeks()

```js
const calcMuscleAvg4weeks = () => {
  const today = currentDate();
  // 4 semanas = 28 dias atras ate ontem
  const end = shiftDateStr(today, -1);
  const start = shiftDateStr(today, -28);
  const raw = calcMuscleVolume(start, end);
  // Dividir por 4 para media semanal
  const avg = {};
  for (const [g, v] of Object.entries(raw)) {
    avg[g] = {
      direct:   +(v.direct   / 4).toFixed(1),
      indirect: +(v.indirect / 4).toFixed(1),
      total:    +(v.total    / 4).toFixed(1),
    };
  }
  return avg;
};

// Helper: deslocar data string YYYY-MM-DD por N dias
const shiftDateStr = (dateStr, days) => {
  const d = new Date(dateStr + "T00:00:00");
  d.setDate(d.getDate() + days);
  return d.toISOString().slice(0, 10);
};
```

### calcFrequencyAlert()

```js
// Retorna array de alertas: { grupo, sets, date }
const calcFrequencyAlert = (startDate, endDate) => {
  const alerts = [];
  const days = treinoData.days || {};
  for (const [date, day] of Object.entries(days)) {
    if (date < startDate || date > endDate) continue;
    const grupoSets = {};
    for (const ex of (day.exercicios || [])) {
      const exInfo = exById(ex.exercicioId);
      const pg = resolvePrimaryGroup(exInfo); // precisa ser extraida de calcMuscleVolume
      const validSets = (ex.series || []).filter(s => (Number(s.reps) || 0) > 0).length;
      if (pg && validSets > 0) grupoSets[pg] = (grupoSets[pg] || 0) + validSets;
    }
    for (const [g, sets] of Object.entries(grupoSets)) {
      if (sets > 11) alerts.push({ grupo: g, sets, date });
    }
  }
  return alerts;
};
```

ATENCAO: `resolvePrimaryGroup` deve ser extraida como funcao nomeada (nao closure interna de calcMuscleVolume) para ser reutilizavel.

---

## Parte 3 — HTML + CSS

### HTML: mudancas no #tmplHistModal (linha ~2648)

Adicionar botao na `.th-tabs`:
```html
<button class="th-tab-btn" id="thTabGrupo" type="button">💪 Por grupo</button>
```

Adicionar painel apos `#thPanelEquip`:
```html
<div class="th-panel" id="thPanelGrupo">
  <div id="thGrupoContent"></div>
</div>
```

### CSS: adicionar nas classes de treino

```css
/* Volume por grupo muscular */
.mg-card {
  background: var(--surface2);
  border-radius: var(--radius);
  padding: 10px 12px;
  margin-bottom: 8px;
  border-left: 3px solid var(--line);
}
.mg-card--ok    { border-left-color: var(--ok); }
.mg-card--over  { border-left-color: var(--bad); }
.mg-card--low   { border-left-color: var(--text3); opacity: 0.7; }

.mg-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 6px;
}
.mg-name  { font-size: 13px; font-weight: 700; color: var(--text); }
.mg-sets  { font-size: 13px; font-weight: 700; color: var(--text); }

.mg-bar-wrap {
  position: relative;
  height: 6px;
  background: var(--line);
  border-radius: 3px;
  margin-bottom: 6px;
  overflow: visible;
}
.mg-bar-fill {
  height: 100%;
  border-radius: 3px;
  background: var(--text3);
  transition: width 0.3s ease;
}
.mg-card--ok   .mg-bar-fill { background: var(--ok); }
.mg-card--over .mg-bar-fill { background: var(--bad); }

.mg-bar-mev {
  position: absolute;
  top: -3px;
  width: 2px;
  height: 12px;
  background: var(--accent);
  border-radius: 1px;
}

.mg-meta {
  display: flex;
  justify-content: space-between;
  font-size: 11px;
  color: var(--text3);
}
.mg-delta-up   { color: var(--ok); }
.mg-delta-down { color: var(--bad); }

.mg-alert-chip {
  display: inline-block;
  background: rgba(248,113,113,0.12);
  color: var(--bad);
  border-radius: 20px;
  padding: 4px 10px;
  font-size: 11px;
  font-weight: 600;
  margin: 4px 4px 0 0;
}
```

---

## Parte 4 — Render e integracao

### renderMuscleVolume()

```js
const renderMuscleVolume = () => {
  const today = currentDate();
  const weekStart = shiftDateStr(today, -6);
  const current = calcMuscleVolume(weekStart, today);
  const avg4 = calcMuscleAvg4weeks();
  const alerts = calcFrequencyAlert(weekStart, today);

  const container = $("#thGrupoContent");
  container.innerHTML = "";

  // Cards por grupo
  let html = "";
  for (const grupo of MUSCLE_ORDER) {
    const v = current[grupo];
    const a = avg4[grupo];
    const lm = MUSCLE_LANDMARKS[grupo];
    const total = v.total;
    const totalRounded = Math.round(total * 2) / 2; // arredonda para .0 ou .5

    let cardClass = "mg-card";
    if (total >= lm.mrv)       cardClass += " mg-card--over";
    else if (total >= lm.mev)  cardClass += " mg-card--ok";
    else                        cardClass += " mg-card--low";

    const barPct = Math.min(total / lm.mrv, 1) * 100;
    const mevPct = Math.min(lm.mev / lm.mrv, 1) * 100;

    const delta = +(total - a.total).toFixed(1);
    let deltaHtml = "";
    if (a.total > 0 || total > 0) {
      if (delta > 0)       deltaHtml = `<span class="mg-delta-up">+${delta} vs media 4sem</span>`;
      else if (delta < 0)  deltaHtml = `<span class="mg-delta-down">${delta} vs media 4sem</span>`;
      else                 deltaHtml = `<span style="color:var(--text3)">= media 4sem</span>`;
    }

    const breakdown = v.direct > 0 || v.indirect > 0
      ? `${v.direct} diretos + ${v.indirect} via compostos`
      : "Nenhuma serie esta semana";

    html += `
      <div class="${cardClass}">
        <div class="mg-header">
          <span class="mg-name">${grupo}</span>
          <span class="mg-sets">${totalRounded} sets</span>
        </div>
        <div class="mg-bar-wrap">
          <div class="mg-bar-fill" style="width:${barPct}%"></div>
          <div class="mg-bar-mev" style="left:${mevPct}%" title="MEV: ${lm.mev}"></div>
        </div>
        <div class="mg-meta">
          <span>${breakdown}</span>
          <span>${deltaHtml}</span>
        </div>
      </div>`;
  }

  // Alertas de frequencia
  if (alerts.length > 0) {
    html += `<div style="margin-top:12px;"><div class="hint" style="margin-bottom:4px;">Alertas de fracionamento:</div>`;
    for (const al of alerts) {
      html += `<span class="mg-alert-chip">⚠ ${al.grupo}: ${al.sets} sets em ${al.date} — considere fracionar em 2x/sem</span>`;
    }
    html += `</div>`;
  }

  // Nota de rodape
  html += `<div style="margin-top:12px;font-size:11px;color:var(--text3);">
    Linha azul na barra = MEV (minimo efetivo). Series via compostos valem 0.5x.
    Baseado em protocolos Lucas Campos / Renaissance Periodization.
  </div>`;

  container.innerHTML = html;
};
```

### switchThTab(): adicionar "Grupo"

```js
const switchThTab = (tab) => {
  ["thTabTreino","thTabEquip","thTabGrupo"]
    .forEach(id => $("#"+id).classList.toggle("active", id === "thTab"+tab));
  ["thPanelTreino","thPanelEquip","thPanelGrupo"]
    .forEach(id => $("#"+id).classList.toggle("active", id === "thPanel"+tab));
  if (tab === "Equip") {
    buildEquipSelect();
    renderEquipmentHistory($("#thEquipSelect").value);
  }
  if (tab === "Grupo") renderMuscleVolume();
};
```

### Listeners em openTmplHistModal()

Adicionar junto com os outros tabs:
```js
$("#thTabGrupo").onclick = () => switchThTab("Grupo");
```

---

## CHECKLIST DE VALIDACAO (antes de /deploy)

- [ ] Parte 0: chips aparecem e ficam ativos/inativos ao tocar
- [ ] Parte 0: chips excluem o grupo primario selecionado
- [ ] Parte 0: ao trocar grupo primario, chips recalculam
- [ ] Parte 0: salvar custom exercise persiste `secundarios` no localStorage
- [ ] Parte 0: rename de custom carrega `secundarios` existentes pre-marcados
- [ ] Parte 1: `EX_SECONDARY` e `MUSCLE_LANDMARKS` definidos antes de qualquer uso
- [ ] Parte 2: `calcMuscleVolume` retorna objeto com todos grupos de MUSCLE_ORDER
- [ ] Parte 2: exercicio custom sem `secundarios` nao quebra (trata como [])
- [ ] Parte 2: exercicio inexistente (exById = null) nao quebra
- [ ] Parte 3: 3a aba aparece no modal de historico
- [ ] Parte 3: scroll funciona no painel Por grupo (modal-body ja tem overflow)
- [ ] Parte 4: cards renderizam sem dados (semana sem treino)
- [ ] Parte 4: barras nao ultrapassam 100% (min/max aplicados)
- [ ] Parte 4: alertas de frequencia so aparecem se > 11 sets num grupo num dia
- [ ] Geral: abas Por treino e Por equipamento continuam funcionando

---

## FORA DO ESCOPO v2.8.0 (fase futura)

- Sistema de prioridades (usuario marca 2 grupos prioritarios)
- Grafico radar/polar
- MEV/MAV configuravel pelo usuario
- Sugestao de treinos para fechar deficit
- Filtro por template na aba Por grupo

---

## INFORMACOES TECNICAS DE CONTEXTO

- Arquivo: `index.html` (~8.135 linhas, single-file PWA)
- Modal: `#tmplHistModal` linha ~2637, funcao `openTmplHistModal` linha ~6830
- `exById()` linha ~5973 — retorna `{id, nome, grupo}` ou null
- `treinoData.days` — objeto com chaves "YYYY-MM-DD"
- `currentDate()` — retorna data selecionada no picker (string "YYYY-MM-DD")
- `customExercises` — array em memoria, salvo em CUSTOM_EX_KEY
- `saveCustomEx()` linha ~6xxx — funcao que persiste custom exercise
- `openExSelCustomRename()` — funcao que abre modal de rename de custom
- CSS vars: `--good` (verde — NAO --ok), `--bad` (vermelho), `--text3` (cinza), `--accent` (roxo), `--surface2`, `--line`, `--radius`
- sw.js CACHE_NAME atual: `kcalix-v11` — fazer bump para `kcalix-v12` no deploy

---

## VERSAO ALVO: v2.8.0
