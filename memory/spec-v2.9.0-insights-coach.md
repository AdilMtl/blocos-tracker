# Spec v2.9.0 + v3.0.0 — Insights Automaticos + Coach Modal

**Status v2.9.0**: IMPLEMENTADO — aguardando deploy (commit 100b848)
**Status v3.0.0**: APROVADO — implementar apos v2.9.0 deployado
**Data**: 2026-03-05 (refinado 2026-03-06)
**Fonte tecnica**: Protocolos Lucas Campos (videos + NotebookLM)
**Pre-requisito obrigatorio**: v2.8.0 deployada
  (calcMuscleVolume, EX_SECONDARY, MUSCLE_LANDMARKS, aba "Por grupo" existindo)

---

## 1. CONTEXTO DO PROJETO

- App: Kcal.ix — single-file PWA (`index.html`, ~8.757 linhas em v2.9.0)
- URL: https://adilmtl.github.io/blocos-tracker/
- Storage treino: `blocos_tracker_treino_v1` -> `treinoData.days[YYYY-MM-DD]`
- Storage custom exercises: `blocos_tracker_custom_exercises_v1`
- Perfil: `blocos_tracker_calc_v6` — contem `goalType` (cut/bulk/recomp/manut)
- sw.js: CACHE_NAME `kcalix-v12` (v2.8.x) → bump para `kcalix-v13` no deploy de v2.9.0

### Funcoes pre-existentes relevantes (disponiveis para usar)
- `exById(id)` — retorna `{id, nome, grupo}` ou null
- `getAllTmplSessions(tmplId, limit)` — sessoes mais recentes
- `calcMuscleVolume(start, end)` — agregado de sets por grupo muscular (v2.8.0)
- `calcMuscleAvg4weeks()` — media semanal das ultimas 4 semanas (v2.8.0)
- `getSecondary(exInfo)` — grupos secundarios de um exercicio (v2.8.0)
- `shiftDateStr(dateStr, days)` — desloca data string YYYY-MM-DD (v2.8.0)
- `currentDate()` — data selecionada no picker (string YYYY-MM-DD)
- `treinoData.days` — objeto com todos os dias salvos
- `MUSCLE_LANDMARKS` — { mev, mav, mrv } por grupo (v2.8.0)
- `MUSCLE_ORDER` — array com ordem de exibicao dos grupos (v2.8.0)

### CSS vars do projeto (ATENCAO: usar --good, nao --ok)
```
--good    verde (positivo/ok)       CORRETO
--bad     vermelho (alerta/excesso)
--text3   cinza (negligenciado/inativo)
--accent  roxo (destaque/interativo)
--surface2, --line, --radius, --font, --text, --text2
```

---

## 2. PRINCIPIOS TECNICOS — PROTOCOLOS LUCAS CAMPOS

### 2.1 Por que funciona para TODOS os 4 objetivos do usuario

O volume de series por grupo muscular e um indicador INDEPENDENTE do objetivo.
O que muda entre cut/bulk/recomp/manutencao e a DIETA, nao o treino.

| Objetivo | O que o volume monitoring faz pelo usuario |
|---|---|
| Cut | Confirma que o volume esta acima do MEV — garante que nao esta perdendo musculo junto com gordura |
| Recomp | Monitora consistencia — estimulo deve ser estavel semana a semana |
| Bulk | Detecta estagnacao de volume — progressao de +20% por ciclo e a referencia |
| Manutencao | MEV literalmente responde "quanto minimo preciso para nao regredir?" |

Conclusao: os insights e o Coach Modal sao igualmente relevantes para qualquer objetivo.
O app nao precisa adaptar os alertas por goalType — o volume e universal.

### 2.2 Contagem de series (padrao Lucas Campos / RP)

- Serie valida = `reps > 0` (independente de carga — proxy para "chegou perto da falha")
- Motor primario: `+1.0 serie` por serie valida
- Sinergista secundario: `+0.5 serie` por serie valida
- Objetivo: contar estimulo, nao tonelagem (kg × reps)

Racional: a tonelagem favorece exercicios pesados e distorce a comparacao entre grupos.
Contar series e o padrao da literatura de hipertrofia (Schoenfeld, Israetel, Lucas Campos).

### 2.3 Volume Landmarks por grupo (valores corrigidos — Lucas Campos)

```js
const MUSCLE_LANDMARKS = {
  "Peito":     { mev: 10, mav: 15, mrv: 22 },
  "Costas":    { mev: 10, mav: 15, mrv: 22 },
  "Quad":      { mev:  8, mav: 14, mrv: 22 },
  "Posterior": { mev:  6, mav: 12, mrv: 20 },
  "Gluteos":   { mev: 15, mav: 20, mrv: 23 },  // Lucas: base de 15 (mais alto de todos)
  "Ombros":    { mev:  6, mav: 12, mrv: 20 },
  "Biceps":    { mev:  6, mav: 12, mrv: 20 },
  "Triceps":   { mev:  6, mav: 12, mrv: 20 },
  "Core":      { mev:  4, mav: 10, mrv: 16 },
};
```

**MEV (Minimo Efetivo de Volume):**
- Tecnicamente = ~20% do volume usado para ganhar (relativo ao historico)
- Na pratica do app: usar valores fixos acima para usuarios sem historico suficiente
- Com 4+ semanas de dados: MEV pode ser calculado como 20% da media de 4 semanas

**MAV (Maximo Adaptativo):** faixa onde o ganho e maximo — 3 a 15-18 series para a maioria

**MRV (Maximo Recuperavel):** Lucas raramente prescreve acima de 20-23 series.
Acima disso: retorno diminui e fadiga sobe exponencialmente.

### 2.4 Rep ranges (Lucas Campos)

- Hipertrofia: 5 a 30 reps — faixa MUITO ampla
- O que importa: PROXIMIDADE A FALHA, nao o numero de reps
- Forca: 5-8 reps com carga alta (adaptacao neural)
- Variar faixas: recomendado para conforto articular e novos estimulos
- Mesmo range por muito tempo: nao proibido, mas risco de sobrecarga tendinosa com cargas altas

### 2.5 Progressao saudavel

- Progressao de carga/reps: a cada 2 semanas para iniciantes, 3 semanas para avancados
- Progressao de volume: +20% por ciclo quando estagnado (referencia Lucas Campos)
- "Quanto mais treinado voce e, menos treinavel voce e." — Lucas Campos

### 2.6 Volume Cycling (NAO e deload tradicional)

Lucas NAO usa o conceito de "deload" como repouso. Usa VOLUME CYCLING:
- Reduzir DRASTICAMENTE o volume (para ~20% do previo = 3-4 series semanais)
- MANTER a intensidade/carga alta (nao reduzir o peso)
- Duracao: 1-2 semanas
- Objetivo: ressensibilizar o musculo para nova progressao
- Analogia: "parar no posto de gasolina — nao para desistir, mas para ir mais longe"

### 2.7 Nivel do usuario

Determinado pelo historico de dados no app (nao requer input adicional):
- `< 3 meses de dados` = iniciante
- `3-12 meses` = intermediario
- `> 12 meses` = avancado

Impacto nos insights:
- Plateau threshold: 2 semanas (iniciante) vs 3 semanas (avancado)
- Linguagem: iniciante recebe orientacao sobre intensidade/dieta; avancado recebe sugestao de volume cycling
- Expectativa de progressao: iniciante progride mais rapido e com mais regularidade

### 2.8 Regra da Prioridade (Push:Pull e desequilibrios)

Lucas NAO estabelece proporcao fixa (ex: 1:1) entre grupos antagonistas.
Defende a Regra da Prioridade: o grupo de maior objetivo do usuario DEVE ter mais volume.

O que monitorar:
- Desequilibrio clinico = quando musculo dominante "atropela" o desenvolvimento do antagonista
- Threshold pratico: ratio > 2.5x E grupo menor abaixo do MEV = sinal real de problema
- NUNCA dizer "treine menos X" — sempre "Y esta abaixo do minimo"

---

## 3. LIMITACOES HONESTAS DO SISTEMA

Esta secao e critica para entender o que o app PODE e NAO PODE fazer.
Importante para guiar comunicacao ao usuario nos insights.

### O que o app consegue (com dados existentes)
- Contar volume total da semana por grupo
- Identificar grupos negligenciados (abaixo MEV)
- Detectar plateau de carga por exercicio
- Comparar volume atual vs media de 4 semanas
- Identificar padroes de rep range monotono
- Detectar desequilibrio entre grupos antagonistas
- Sinalizar fadiga sistemica (queda de forca em multiplos exercicios)

### O que o app NAO consegue (sem dado adicional do usuario)
- Saber se as series chegaram perto da falha (RIR/RPE)
- Detectar pre-fadiga por ordem de exercicios na sessao
- Diferenciar serie "facil" de serie "dura"
- Saber se a queda de carga foi intencional ou por fadiga

### Como o usuario valida se o volume esta funcionando

As duas abas se complementam:
- **Aba "Por grupo"**: mostra se voce esta treinando o SUFICIENTE (mapa do volume)
- **Aba "Por equipamento"**: mostra se esse treino esta PRODUZINDO resultado (progressao de carga)

Interpretacao correta:
```
Volume OK + progressao de carga subindo = tudo certo
Volume OK + carga estagnada = problema de qualidade (series longe da falha? recuperacao? dieta?)
Volume baixo + carga estagnada = provavelmente falta de volume
Volume alto + carga caindo = sinal de fadiga acumulada → volume cycling
```

Esta logica deve aparecer na descricao do Coach Modal (pagina de fundamentos).

---

## 4. JORNADA DO USUARIO — 5 CENAS

### Cena 1: Criando exercicio customizado (ja existente em v2.8.0)

Ao criar "Supino maquina (minha academia)", usuario escolhe:
- Grupo primario: Peito
- Grupos secundarios (opcional, chips de toggle): Triceps, Ombros

App agora sabe que esse exercicio recruta esses musculos tambem.
Se nao marcar nenhum chip: funciona igual a antes (sem secundarios).

### Cena 2: Abrindo historico de treino

Modal abre normalmente. Agora tem 3 abas:
"Por treino" | "Por equipamento" | "Por grupo"

### Cena 3: A visao de volume muscular (v2.8.0)

Aba "Por grupo" mostra cards por musculo:
```
Peito              15.5 sets  ████████████████░░░░░
                              MEV | MRV
12 diretos + 3.5 via compostos    +2.5 vs media 4 sem
```
Paleta: cinza (< MEV) / verde (MEV-MRV) / vermelho (> MRV)
Linha na barra = MEV. Largura proporcional ao MRV.

### Cena 4: Insights automaticos (v2.9.0) — DESIGN FINAL

DIFERENTE da spec original: os insights NAO ficam numa lista no topo da aba.
Ficam INLINE no rodape de cada `.mg-card`, como chips clicaveis.

```
┌─────────────────────────────────────────────────┐
│ Peito                                  15.5 sets │
│ ████████████████░░░   MEV 10 / MRV 22            │
│ 12 diretos + 3.5 via compostos  +2.5 vs 4sem     │
│ ─────────────────────────────────────────────    │
│ [✅ Volume ideal] [⚠ Supino sem progressao 3sem] │
└─────────────────────────────────────────────────┘
```

Chip positivo (verde): aparece quando tudo ok — "Volume ideal"
Chips de alerta (laranja/roxo): clicaveis, expandem para mostrar detalhe e acao

### Cena 5: O insight que o usuario nao via antes

Exemplos reais do que o app pode revelar:
- "Posterior de coxa abaixo do MEV ha 3 semanas — você achava que treinava perna, mas fazia so quad"
- "Biceps em 9 sets sem fazer nenhuma rosca — vieram todos via puxadas e remadas"
- "Supino sem progressao ha 3 semanas — hora de virar a pagina"
- "Costas com volume 3x menor que Peito — casa com paredes fortes e teto fraco"

---

## 5. v2.9.0 — INSIGHTS AUTOMATICOS

### 5.1 Design final (difere da spec original)

Insights sao gerados por funcao `buildInsightsByGroup(currentVol)`.
Retorna objeto `{ [grupoKey]: [array de insights] }` — organizado por grupo.

Cada insight e um chip no rodape do `.mg-card` correspondente.
Ha tambem um chip POSITIVO quando o grupo esta dentro da faixa ideal.

Classes de chip:
- `.mg-chip--ok`  → verde  → "Volume ideal" (positivo)
- `.mg-chip--w`   → laranja → warning (plateau, desequilibrio, negligencia)
- `.mg-chip--i`   → roxo   → info (rep monotonia, sugestao de cycling)

Toggle de expand/collapse: `window.mgChipToggle(chip, detailId)`
O detalhe expandido aparece como `<div id="[detailId]">` logo apos o chip.

### 5.2 Funcao getUserLevel()

```js
const getUserLevel = () => {
  const dates = Object.keys(treinoData.days || {}).sort();
  if (dates.length === 0) return "iniciante";
  const first = new Date(dates[0] + "T00:00:00");
  const now = new Date();
  const months = (now - first) / (1000 * 60 * 60 * 24 * 30);
  if (months < 3)  return "iniciante";
  if (months < 12) return "intermediario";
  return "avancado";
};
```

### 5.3 Funcao detectPlateaus()

```
Algoritmo:
1. Para cada exercicio em treinoData.days (todas as datas):
   a. Coletar: data + max carga da sessao para esse exercicio
   b. Ordenar por data crescente
   c. Agrupar por semana (ISO week ou janela de 7 dias)
   d. Pegar max carga de cada semana

Threshold de plateau (sem progressao = carga igual ou menor):
  - getUserLevel() === "iniciante": 2 semanas sem progressao
  - getUserLevel() !== "iniciante": 3 semanas sem progressao

Condicao adicional: exercicio deve ter sido feito em pelo menos
N sessoes no periodo (evitar alertas com dados insuficientes):
  - Iniciante: minimo 2 sessoes no periodo
  - Avancado: minimo 3 sessoes no periodo

Retorno por exercicio:
{ exId, exNome, grupo, semanas, maxCarga, nivel }
```

**Linguagem dos chips:**
```
Iniciante, 2 semanas:
titulo: "[ExNome] sem progressao ha 2 semanas"
detalhe: "Para iniciantes, a progressao deve ser quase linear.
As series estao chegando perto da falha? A dieta esta adequada para o seu objetivo?
Tente adicionar 1-2kg na proxima sessao."

Avancado, 3 semanas:
titulo: "[ExNome] sem progressao ha 3 semanas"
detalhe: "Normal para o seu nivel de treinamento — 'quanto mais treinado, menos treinavel'.
Considere um ciclo de reducao: 3-4 series/semana por 2 semanas mantendo a carga,
depois retome com novo estimulo."
```

### 5.4 Funcao detectVolumeCyclingNeed()

```
Algoritmo — 2 gatilhos independentes:

GATILHO A: Volume cronicamente alto
  Para cada grupo em MUSCLE_ORDER:
    Calcular volume de cada uma das ultimas 6 semanas (calcMuscleVolume semana a semana)
    Se 4 das ultimas 6 semanas ficaram ACIMA do MAV: gatilho

GATILHO B: Queda de forca cruzada (fadiga sistemica)
  Para um mesmo grupo:
    Comparar max carga desta semana vs semana anterior para cada exercicio
    Se 2+ exercicios DIFERENTES do mesmo grupo tiveram queda de carga: gatilho
    (queda = max carga atual < max carga semana anterior)

Seguranca: nao disparar se usuario tem < 6 semanas de historico (dados insuficientes)

Retorno por grupo:
{ grupo, gatilho: "volume_alto" | "forca_caindo", semanas: N, sugestao: "3-4 series" }
```

**Linguagem dos chips:**
```
Gatilho A:
titulo: "[Grupo] acima do volume ideal por [N] semanas"
detalhe: "Seu motor precisa parar no posto — nao para desistir, mas para ir mais longe.
Sugestao: reduza para 3-4 series/semana por 1-2 semanas. MANTENHA a carga.
Depois retome o volume normal — voce vai notar a diferenca."

Gatilho B:
titulo: "Queda de forca em [N] exercicios de [Grupo]"
detalhe: "Sinal de fadiga acumulada — o musculo nao esta se recuperando entre as sessoes.
Considere 1 semana de manutencao: 3-4 series com a mesma carga, sem aumentar o volume."
```

### 5.5 Funcao detectRepMonotony()

```
Algoritmo:
Para cada exercicio com 4+ sessoes no historico (ultimas 8 semanas):
  1. Coletar media de reps de cada sessao (media das series validas)
  2. Calcular desvio padrao das medias das ultimas 4 sessoes
  3. Se desvio padrao < 2.0: mesma faixa ha tempo = monotonia

Sub-caso critico (risco articular):
  Se media de reps < 8 em 6+ sessoes consecutivas:
  → cargas altas por muito tempo = risco de sobrecarga tendinosa

Retorno:
{ exId, exNome, mediaReps, sessoes, tipo: "monotonia" | "carga_alta_longa" }
```

**Linguagem:**
```
Monotonia simples:
titulo: "[ExNome] sempre ~[X] reps ha [N] sessoes"
detalhe: "Variar a faixa de reps pode trazer novos estimulos e melhorar o conforto articular.
Para hipertrofia, qualquer faixa entre 5 e 30 reps funciona — o que importa e chegar perto da falha.
Experimente uma fase com reps mais altas ou mais baixas."

Carga alta longa:
titulo: "[ExNome] com cargas pesadas ha 6+ sessoes"
detalhe: "Cargas altas por muito tempo podem sobrecarregar tendoes e articulacoes.
Considere uma fase com reps mais altas (12-15) por algumas semanas para dar descanso articular."
```

### 5.6 Funcao detectMuscleImbalance()

```
Algoritmo:
Verificar 3 pares de grupos antagonistas na semana atual:
  Par 1: Peito vs Costas
  Par 2: Quad vs Posterior
  Par 3: Biceps vs Triceps

Para cada par:
  volA = volume total do grupo A
  volB = volume total do grupo B
  Se volA > volB: ratio = volA / volB
  Condicao de alerta: ratio > 2.5 E volB < MEV do grupo B

Seguranca (Regra da Prioridade):
  - Alertar apenas se o grupo MENOR esta abaixo do MEV
  - Se ambos estao acima do MEV, nao e desequilibrio clinico — e escolha do usuario
  - NUNCA dizer "treine menos [A]" — sempre "treine mais [B]"

Retorno:
{ grupoA, volA, grupoB, volB, ratio }
```

**Linguagem:**
```
titulo: "[GrupoA] com [X]x mais volume que [GrupoB]"
detalhe: "Treinar de forma desigual e como construir uma casa com paredes fortes e teto fraco.
[GrupoB] abaixo do minimo pode limitar o proprio desenvolvimento de [GrupoA] e afetar a postura.
Nao precisa treinar menos [GrupoA] — so garantir que [GrupoB] atinja pelo menos [MEV] series/semana."
```

### 5.7 Funcao detectChronicLowVolume()

```
Algoritmo:
Para cada grupo em MUSCLE_ORDER:
  Calcular volume das ultimas 4 semanas (semana a semana)
  Contar quantas semanas ficou abaixo do MEV
  Se 3+ das 4 semanas abaixo do MEV: negligencia cronica

Diferenca do card visual:
  Card mostra snapshot da semana atual (cinza/verde/vermelho)
  Este insight detecta PADRAO — 3 semanas consecutivas e mais urgente que 1 semana
```

**Linguagem por grupo (sugestoes especificas):**
```
"[Grupo] abaixo do minimo por [N] das ultimas 4 semanas."

+ sugestao especifica por grupo:
  Posterior de coxa: "1 mesa flexora + 1 stiff por semana ja coloca voce na faixa."
  Gluteos:   "Hip thrust tem o maior retorno para gluteos. 3 series 2x/semana resolve."
  Core:      "2-3 series de prancha ou crunch ao final de qualquer treino e suficiente."
  Biceps:    "Ja vem de costas — verifique se suas remadas estao sendo contabilizadas corretamente."
  Triceps:   "Ja vem do peito — verifique se seus supinos estao sendo contabilizados."
  Posterior coxa: "Stiff e cadeira flexora — idealmente no inicio da sessao de pernas."
```

### 5.8 Funcao buildInsightsByGroup(currentVol)

Funcao central que organiza todos os insights por grupo:

```js
const buildInsightsByGroup = (currentVol) => {
  const level = getUserLevel();
  const plateaus = detectPlateaus();
  const cycling = detectVolumeCyclingNeed();
  const monotony = detectRepMonotony();
  const imbalance = detectMuscleImbalance();
  const chronic = detectChronicLowVolume();

  // Resultado: { "Peito": [insightObj, ...], "Costas": [...], ... }
  const byGroup = {};
  MUSCLE_ORDER.forEach(g => { byGroup[g] = []; });

  // Plateau: associar ao grupo do exercicio
  plateaus.forEach(p => {
    if (byGroup[p.grupo]) byGroup[p.grupo].push({
      tipo: "plateau", nivel: "w",
      icone: "⚠",
      titulo: `${p.exNome} sem progressao ha ${p.semanas} semanas`,
      detalhe: /* linguagem por nivel */ buildPlateauText(p, level),
      acao: "Ver historico",
      acaoFn: `switchThTab('Equip'); buildEquipSelect(); document.getElementById('thEquipSelect').value='${p.exId}'; renderEquipmentHistory('${p.exId}');`
    });
  });

  // Volume cycling: associar ao grupo
  cycling.forEach(c => {
    if (byGroup[c.grupo]) byGroup[c.grupo].push({
      tipo: "cycling", nivel: "w",
      icone: "⛽",
      titulo: c.gatilho === "volume_alto"
        ? `${c.grupo} acima do ideal por ${c.semanas} semanas`
        : `Queda de forca em ${c.grupo}`,
      detalhe: buildCyclingText(c),
      acao: null
    });
  });

  // Imbalance: associar ao grupo MENOR (que precisa de atencao)
  imbalance.forEach(im => {
    if (byGroup[im.grupoB]) byGroup[im.grupoB].push({
      tipo: "imbalance", nivel: "w",
      icone: "⚖",
      titulo: `${im.grupoA} com ${im.ratio.toFixed(1)}x mais volume`,
      detalhe: buildImbalanceText(im),
      acao: null
    });
  });

  // Chronic low: associar ao grupo
  chronic.forEach(c => {
    if (byGroup[c.grupo]) byGroup[c.grupo].push({
      tipo: "chronic", nivel: "w",
      icone: "📉",
      titulo: `${c.grupo} abaixo do minimo por ${c.semanasAbaixo} semanas`,
      detalhe: buildChronicText(c),
      acao: null
    });
  });

  // Rep monotony: associar ao grupo do exercicio
  monotony.forEach(m => {
    const exInfo = exById(m.exId);
    const grp = exInfo ? resolveAnalyticsGroup(exInfo) : null;
    if (grp && byGroup[grp]) byGroup[grp].push({
      tipo: "monotony", nivel: "i",
      icone: "ℹ",
      titulo: m.tipo === "carga_alta_longa"
        ? `${m.exNome}: cargas pesadas ha 6+ sessoes`
        : `${m.exNome}: sempre ~${Math.round(m.mediaReps)} reps`,
      detalhe: buildMonotonyText(m),
      acao: null
    });
  });

  // Chip positivo: grupos sem nenhum insight e dentro da faixa
  MUSCLE_ORDER.forEach(g => {
    const v = currentVol[g];
    const lm = MUSCLE_LANDMARKS[g];
    if (byGroup[g].length === 0 && v && v.total >= lm.mev && v.total <= lm.mrv) {
      byGroup[g].push({
        tipo: "ok", nivel: "ok",
        icone: "✅",
        titulo: "Volume ideal esta semana",
        detalhe: `${g} na faixa de ${lm.mev} a ${lm.mrv} series semanais.`,
        acao: null
      });
    }
  });

  return byGroup;
};
```

### 5.9 Toggle de chips — window.mgChipToggle

```js
window.mgChipToggle = (chipEl, detailId) => {
  const detail = document.getElementById(detailId);
  if (!detail) return;
  const isOpen = detail.style.display !== "none";
  detail.style.display = isOpen ? "none" : "block";
  chipEl.classList.toggle("mg-chip--open", !isOpen);
};
```

Chamada inline no HTML do chip:
```html
<span class="mg-chip mg-chip--w" onclick="mgChipToggle(this, 'ins_plateau_supino_reto')">
  ⚠ Supino sem progressao ha 3 semanas
</span>
<div id="ins_plateau_supino_reto" style="display:none;" class="mg-chip-detail">
  Texto do detalhe + botao de acao
</div>
```

### 5.10 CSS das classes de insight (v2.9.0)

```css
/* Chips de insight no rodape dos mg-cards */
.mg-chip-row {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin-top: 8px;
  padding-top: 8px;
  border-top: 1px solid var(--line);
}
.mg-chip {
  display: inline-flex;
  align-items: center;
  gap: 4px;
  padding: 4px 10px;
  border-radius: 20px;
  font-size: 11px;
  font-weight: 600;
  cursor: pointer;
  border: 1px solid transparent;
  transition: opacity 0.15s;
}
.mg-chip--ok { background: rgba(var(--good-rgb), 0.12); color: var(--good); border-color: rgba(var(--good-rgb), 0.25); }
.mg-chip--w  { background: rgba(248,113,113, 0.12);    color: var(--bad);  border-color: rgba(248,113,113, 0.25); }
.mg-chip--i  { background: rgba(var(--accent-rgb),0.12); color: var(--accent); border-color: rgba(var(--accent-rgb),0.25); }
.mg-chip-detail {
  margin-top: 8px;
  font-size: 12px;
  color: var(--text2);
  line-height: 1.5;
  padding: 8px 10px;
  background: var(--surface2);
  border-radius: var(--radius-xs);
}
.mg-chip-action {
  display: inline-block;
  margin-top: 8px;
  font-size: 11px;
  font-weight: 700;
  color: var(--accent);
  background: none;
  border: 1px solid var(--accent);
  border-radius: 20px;
  padding: 3px 10px;
  cursor: pointer;
}
```

### 5.11 Integracao em renderMuscleVolume()

```js
// Ao final de renderMuscleVolume(), depois de construir os cards:
const insightsByGroup = buildInsightsByGroup(current);

MUSCLE_ORDER.forEach(grupo => {
  const insights = insightsByGroup[grupo] || [];
  if (insights.length === 0) return;

  const card = container.querySelector(`[data-grupo="${grupo}"]`);
  if (!card) return;

  const row = document.createElement("div");
  row.className = "mg-chip-row";

  insights.forEach((ins, i) => {
    const detailId = `mgd_${grupo.replace(/\W/g,"")}_${i}`;
    const chip = document.createElement("span");
    chip.className = `mg-chip mg-chip--${ins.nivel}`;
    chip.textContent = `${ins.icone} ${ins.titulo}`;
    chip.setAttribute("onclick", `mgChipToggle(this, '${detailId}')`);
    row.appendChild(chip);

    const detail = document.createElement("div");
    detail.id = detailId;
    detail.className = "mg-chip-detail";
    detail.style.display = "none";
    detail.innerHTML = ins.detalhe;
    if (ins.acao && ins.acaoFn) {
      detail.innerHTML += `<div><button class="mg-chip-action" type="button"
        onclick="${ins.acaoFn}">${ins.acao}</button></div>`;
    }
    row.appendChild(detail);
  });

  card.appendChild(row);
});
```

---

## 6. Plano de implementacao v2.9.0 (4 partes)

**Parte 1: Helpers de deteccao**
Adicionar apos `calcMuscleAvg4weeks()`:
- `getUserLevel()`
- `detectPlateaus()`
- `detectVolumeCyclingNeed()`
- `detectRepMonotony()`
- `detectMuscleImbalance()`
- `detectChronicLowVolume()`
- Funcoes de texto: `buildPlateauText`, `buildCyclingText`, `buildImbalanceText`, `buildChronicText`, `buildMonotonyText`

**Parte 2: Funcao central**
- `buildInsightsByGroup(currentVol)` — retorna insights organizados por grupo
- `window.mgChipToggle(chip, detailId)` — exposto globalmente

**Parte 3: CSS**
Adicionar nas classes de treino:
- `.mg-chip-row`, `.mg-chip`, `.mg-chip--ok`, `.mg-chip--w`, `.mg-chip--i`
- `.mg-chip-detail`, `.mg-chip-action`

**Parte 4: Integracao em renderMuscleVolume()**
- Chamar `buildInsightsByGroup(current)` apos construir os cards
- Adicionar `.mg-chip-row` no rodape de cada `.mg-card`
- Adicionar atributo `data-grupo` nos cards para seletores

---

## 7. CHECKLIST DE VALIDACAO — v2.9.0

### Funcoes de deteccao
- [ ] getUserLevel retorna "iniciante" para usuario sem historico
- [ ] detectPlateaus nao quebra se exercicio tem apenas 1 sessao
- [ ] detectPlateaus usa threshold correto por nivel (2sem vs 3sem)
- [ ] detectVolumeCyclingNeed nao dispara com < 6 semanas de historico
- [ ] detectVolumeCyclingNeed gatilho B exige 2+ exercicios diferentes (nao 2 series do mesmo)
- [ ] detectMuscleImbalance nao alerta se AMBOS os grupos estao acima do MEV
- [ ] detectMuscleImbalance nunca sugere "treine menos X"
- [ ] detectChronicLowVolume conta semanas corretamente (3 das 4 ultimas)
- [ ] detectRepMonotony nao dispara com < 4 sessoes de historico
- [ ] Todas as funcoes retornam array vazio (nao null/undefined) quando sem dados

### Insights nos cards
- [ ] Chips aparecem no rodape do card correto (grupo certo)
- [ ] Chip positivo aparece quando grupo esta na faixa e sem alertas
- [ ] Chip expande/colapsa ao tocar (mgChipToggle funciona)
- [ ] Detalhe expandido tem texto legivel em 375px
- [ ] Botao "Ver historico" leva para aba Por equipamento com filtro correto
- [ ] Cards de grupo com total = 0 nao quebram (sem insights de plateau etc)

### Geral
- [ ] renderMuscleVolume nao quebra com treinoData.days vazio
- [ ] Abas "Por treino" e "Por equipamento" continuam funcionando
- [ ] Sem erros de console em nenhum cenario
- [ ] sw.js bumped para kcalix-v13

---

## 8. v3.0.0 — COACH MODAL

### 8.1 O que e

Modal educativo acessivel pela aba Mais e pelo botao "?" nos cards de grupo muscular.
Conteudo: literatura simplificada do Lucas Campos em 5 paginas.
Linguagem: sem jargao, usando as metaforas canonicas do Lucas.

### 8.2 Acesso ao modal

1. Botao "Guia de Treino" ou "Coach" na aba Mais (grid de acoes, novo card)
2. Botao "?" no header de cada `.mg-card` na aba "Por grupo"
   → `openCoachModal(grupoKey)` — abre direto na pagina do grupo

### 8.3 Estrutura HTML

```html
<!-- Apos o ultimo modal existente, antes do /app -->
<div class="modal-overlay" id="coachOverlay" style="z-index:330;"></div>
<div class="modal-sheet modal-full" id="coachModal" style="z-index:331;">
  <div class="modal-handle"></div>
  <div class="modal-header">
    <b>Guia de Treino</b>
    <button class="modal-close" id="coachClose" type="button">✕</button>
  </div>
  <div class="modal-body">
    <div class="coach-tabs" id="coachTabs"></div>
    <div id="coachContent"></div>
    <div style="height:40px;"></div>
  </div>
</div>
```

Z-index: overlay 330, modal 331 (proximo acima dos existentes em v2.9.0)

### 8.4 Paginas do Coach (5)

**Pagina 1: Fundamentos**
- O que sao series validas (perto da falha, RIR intuitivo)
- O que e MEV, MAV, MRV — com tabela visual por grupo
- Por que contar series (nao kg × reps)
- Como as duas abas se complementam (Por grupo = mapa, Por equipamento = validacao)
- A limitacao honesta: o app nao detecta qualidade da serie, so quantidade

**Pagina 2: Progressao e Platô**
- 3 formas de progredir: carga, reps, volume
- Expectativa de progressao por nivel
- Citacao: "Quanto mais treinado voce e, menos treinavel voce e" — Lucas Campos
- O que e um platô — metafora das 10 paginas
- Como resolver: carga, rep range, volume cycling

**Pagina 3: Volume Cycling**
- Por que NAO e deload — e volume cycling
- Metafora do posto de gasolina
- Como fazer: reduzir para 3-4 series, MANTER a carga, 1-2 semanas
- Quando fazer: sinais do app (queda de forca, volume alto por 4+ semanas)
- O que esperar depois: ressensibilizacao = melhor resposta ao treino

**Pagina 4: Equilibrio Muscular**
- Metafora da casa (paredes + teto)
- Pares antagonistas: Peito/Costas, Quad/Posterior, Biceps/Triceps
- Regra da Prioridade: nao e obrigatorio 1:1, mas antagonista nao pode ficar abaixo do MEV
- Volume indireto: o que compostos contribuem para cada grupo
- Funciona para todos os objetivos (cut/bulk/recomp/manut)

**Pagina 5: Por Grupo Muscular**
- Botoes para cada grupo (mesma ordem do MUSCLE_ORDER)
- Ao selecionar: MEV/MAV/MRV, exercicios de maior retorno, frequencia ideal, dica especial
- Entrada direta: `openCoachModal("Gluteos")` mostra pagina 5 ja no grupo Gluteos

Guia por grupo (dados para cada):
```
Peito:
  mev:10 mav:15 mrv:22 | freq: 2x/sem ideal
  exercicios: supino reto, supino inclinado, crossover, peck deck
  dica: "Volume indireto de triceps e ombros ja conta — calcule o total antes de adicionar mais series"

Costas:
  mev:10 mav:15 mrv:22 | freq: 2x/sem ideal
  exercicios: puxada frontal, remada curvada, remada unilateral, barra fixa
  dica: "Cada serie de costas traz 0.5 serie de biceps de brinde — verifique o total antes de adicionar roscas"

Quad:
  mev:8 mav:14 mrv:22 | freq: 1-2x/sem
  exercicios: agachamento, leg press, cadeira extensora, hack squat
  dica: "Pernas no inicio do treino — quando mais descansado, maior a qualidade das series"

Posterior de coxa:
  mev:6 mav:12 mrv:20 | freq: 1-2x/sem
  exercicios: mesa flexora, stiff, cadeira flexora, stiff romeno
  dica: "O grupo mais negligenciado das pernas. Stiff e o de maior retorno por serie"

Gluteos:
  mev:15 mav:20 mrv:23 | freq: 2-3x/sem (aguenta volume alto)
  exercicios: hip thrust, stiff romeno, elevacao pelvica, gluteo no cabo
  dica: "MEV de 15 series — mais alto que qualquer outro grupo. Volume baixo nao funciona para gluteos"

Ombros:
  mev:6 mav:12 mrv:20 | freq: 2x/sem
  exercicios: elevacao lateral, desenvolvimento, face pull, crucifixo inverso
  dica: "Elevacao lateral e o exercicio mais eficiente por serie para deltoides medial"

Biceps:
  mev:6 mav:12 mrv:20 | freq: 2x/sem
  exercicios: rosca direta, rosca alternada, rosca martelo, rosca scott
  dica: "Muito volume ja vem de costas (0.5x por serie). Some tudo antes de adicionar roscas isoladas"

Triceps:
  mev:6 mav:12 mrv:20 | freq: 2x/sem
  exercicios: triceps pulley, triceps testa, paralelas, triceps frances
  dica: "Muito volume ja vem do peito e ombros. Some tudo antes de adicionar mais isolados"

Core:
  mev:4 mav:10 mrv:16 | freq: 2-3x/sem (recupera rapido)
  exercicios: prancha, abdominal crunch, roda abdominal, rotacao russa
  dica: "Core recupera rapido — pode treinar ao final de qualquer sessao sem prejuizo"
```

### 8.5 CSS do Coach Modal

```css
.coach-tabs {
  display: flex;
  gap: 6px;
  flex-wrap: wrap;
  margin-bottom: 14px;
}
.coach-tab-btn {
  padding: 6px 12px;
  border-radius: 20px;
  border: 1px solid var(--line);
  font-size: 12px;
  font-weight: 600;
  background: transparent;
  color: var(--text2);
  cursor: pointer;
}
.coach-tab-btn.active {
  background: var(--accent);
  color: #fff;
  border-color: var(--accent);
}
.coach-section { margin-bottom: 20px; }
.coach-headline {
  font-size: 15px;
  font-weight: 700;
  color: var(--text);
  margin-bottom: 8px;
}
.coach-body {
  font-size: 13px;
  color: var(--text2);
  line-height: 1.6;
}
.coach-quote {
  border-left: 3px solid var(--accent);
  padding: 8px 12px;
  margin: 12px 0;
  font-size: 13px;
  font-style: italic;
  color: var(--text2);
  background: var(--surface2);
  border-radius: 0 var(--radius-xs) var(--radius-xs) 0;
}
.coach-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 12px;
  margin-top: 8px;
}
.coach-table th {
  text-align: left;
  color: var(--text3);
  font-weight: 600;
  padding: 4px 8px;
  border-bottom: 1px solid var(--line);
}
.coach-table td {
  padding: 6px 8px;
  border-bottom: 1px solid rgba(255,255,255,0.04);
}
.coach-grupo-btn {
  padding: 6px 14px;
  border-radius: 20px;
  border: 1px solid var(--line);
  font-size: 12px;
  font-weight: 600;
  background: transparent;
  color: var(--text2);
  cursor: pointer;
  margin: 3px;
}
.coach-grupo-btn.active {
  background: var(--surface2);
  border-color: var(--accent);
  color: var(--text);
}
```

### 8.6 JS: openCoachModal / closeCoachModal

```js
const openCoachModal = (grupoInicial) => {
  renderCoachTabs();
  renderCoachPage("fundamentos");
  if (grupoInicial) {
    renderCoachPage("por-grupo", grupoInicial);
  }
  $("#coachOverlay").classList.add("open");
  $("#coachModal").classList.add("open");
};

const closeCoachModal = () => {
  $("#coachOverlay").classList.remove("open");
  $("#coachModal").classList.remove("open");
};
```

### 8.7 Plano de implementacao v3.0.0 (3 partes)

**Parte 1: Modal + CSS**
- HTML do #coachModal + #coachOverlay (antes do fechamento de #app)
- CSS das classes `.coach-*`

**Parte 2: Conteudo**
- Objeto `COACH_PAGES` com as 5 paginas como funcoes geradoras de HTML
- `renderCoachPage(pageId, grupoKey)` — renderiza no #coachContent
- `renderCoachTabs()` — renderiza tabs de navegacao

**Parte 3: Integracao**
- Botao na aba Mais (card no grid de acoes, estilo home-action-card)
- Botao "?" em cada `.mg-card` com `onclick="openCoachModal('${grupo}')"`
- `openCoachModal()` / `closeCoachModal()` + listeners
- sw.js: bump para `kcalix-v14` no deploy de v3.0.0

---

## 9. CHECKLIST DE VALIDACAO — v3.0.0

- [ ] Modal abre sem travar scroll do app
- [ ] Botao "?" nos mg-cards abre na pagina correta (guia do grupo)
- [ ] Tabs do coach navegam sem erros
- [ ] Pagina "Por grupo" exibe dados corretos para cada grupo
- [ ] Tabela de MEV/MAV/MRV renderiza corretamente em 375px
- [ ] Botao na aba Mais aparece e abre o modal
- [ ] Fechar modal libera scroll do fundo
- [ ] sw.js bumped para kcalix-v14

---

## 10. METAFORAS DO LUCAS CAMPOS (usar exatamente estas)

```
PLATEAU — "virar a pagina":
"Seu corpo se acostumou com o desafio atual. E como ler as mesmas
10 paginas de um livro varias vezes — voce nao vai aprender nada
novo ate virar a pagina e aumentar o desafio."

VOLUME CYCLING — "parar no posto":
"Pense no seu treino como uma viagem de carro: de vez em quando
precisamos parar no posto, nao para abandonar a viagem, mas para
garantir que o motor nao quebre e possamos chegar mais longe."

DESEQUILIBRIO — "paredes e teto":
"Treinar o corpo de forma desigual e como construir uma casa com
paredes fortes e um teto fraco. Para ter um shape bonito e saudavel,
precisamos distribuir o trabalho de forma que nenhuma parte fique
sobrecarregada ou esquecida."

NIVEL DE TREINAMENTO:
"Quanto mais treinado voce e, menos treinavel voce e."
```

---

## 11. FORA DO ESCOPO (v3.x+)

- Sistema de prioridades: usuario marca grupos prioritarios → app ajusta thresholds
- MEV/MAV/MRV configuravel pelo usuario
- Sugestao automatica de treino para fechar deficit de volume
- Grafico radar de distribuicao muscular
- Correlacao volume de treino × progresso corporal (peso, medidas)
- Pre-fadiga por ordem de exercicios na sessao
- RPE/RIR input por serie (qualidade da serie)
- Comparacao anonimizada entre usuarios

---

## 12. RESUMO EXECUTIVO PARA NOVA SESSAO

### Contexto geral
Kcal.ix e um PWA single-file. Todo o app vive em index.html.
v2.8.0 (deployada) adicionou analytics de volume muscular com calcMuscleVolume,
MUSCLE_LANDMARKS, EX_SECONDARY e a aba "Por grupo" no modal de historico.

### v2.9.0 (implementado, aguarda deploy)
Adiciona INSIGHTS AUTOMATICOS inline nos cards de grupo muscular.
5 detectores baseados nos protocolos Lucas Campos:
  1. Plateau de carga (2sem iniciante / 3sem avancado)
  2. Necessidade de Volume Cycling (volume alto 4sem OU queda de forca cruzada)
  3. Rep Range monotonia (std dev < 2 em 4 sessoes)
  4. Desequilibrio muscular (ratio > 2.5x entre antagonistas, menor abaixo MEV)
  5. Negligencia cronica (3 das ultimas 4 semanas abaixo do MEV)

Design final: chips no RODAPE de cada mg-card (nao lista global no topo).
Chip positivo "Volume ideal" quando tudo ok.
Toggle via window.mgChipToggle(chip, detailId).
Classes: .mg-chip--ok / .mg-chip--w / .mg-chip--i

### v3.0.0 (aprovado, implementar apos v2.9.0 deployada)
Adiciona COACH MODAL com guia educativo em 5 paginas.
Acessivel pela aba Mais e pelo botao "?" em cada card de grupo.
Conteudo: fundamentos de volume, progressao/platô, volume cycling,
equilibrio muscular, guia detalhado por grupo muscular.
Linguagem: metaforas do Lucas (virar pagina / posto / casa).
Z-index: overlay 330 / modal 331.

### Principio mais importante
NUNCA dizer "treine menos X" — sempre "Y esta abaixo do minimo".
Isso e a Regra da Prioridade do Lucas Campos.
O volume e universal: funciona para cut, bulk, recomp e manutencao igualmente.
