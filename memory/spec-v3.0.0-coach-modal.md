# Spec v3.0.0 — Coach Modal (Guia de Treino)

**Status**: APROVADO — implementar APOS v2.9.0 deployado
**Data**: 2026-03-06
**Fonte tecnica**: Protocolos Lucas Campos (videos + NotebookLM)
**Pre-requisito absoluto**: v2.9.0 deployada
  (insights automaticos nos cards, buildInsightsByGroup, mg-chip-row existindo)

---

## 1. CONTEXTO DO PROJETO

- App: Kcal.ix — single-file PWA (`index.html`, ~8.757 linhas apos v2.9.0)
- URL: https://adilmtl.github.io/blocos-tracker/
- sw.js CACHE_NAME em v2.9.0: `kcalix-v13` → bump para `kcalix-v14` no deploy de v3.0.0

### Estado do app ao implementar v3.0.0

Funcoes e estruturas ja existentes que o Coach vai usar:
- `MUSCLE_LANDMARKS` — { mev, mav, mrv } por grupo
- `MUSCLE_ORDER` — array com ordem dos grupos musculares
- `calcMuscleVolume(start, end)` — volume semanal por grupo
- `renderMuscleVolume()` — renderiza aba "Por grupo"
- `switchThTab(tab)` — troca aba no #tmplHistModal
- `.mg-card` com `data-grupo` — cards de volume ja existentes
- CSS vars: `--good` (verde), `--bad` (vermelho), `--text3` (cinza), `--accent` (roxo),
            `--surface2`, `--line`, `--radius`, `--radius-xs`, `--font`, `--text`, `--text2`

Z-index hierarchy ate v2.9.0:
- nav: 100
- modais padrao: 300/301
- tmplHistModal: 300 (padrao)
- customExModal: 327
- exSelectorModal: 329
- **coachModal: 331 / coachOverlay: 330** ← proximo disponivel

---

## 2. O QUE E O COACH MODAL

Um modal educativo que traduz os protocolos do Lucas Campos em linguagem acessivel.
Nao e um chatbot. Nao e um consultor. E um guia de referencia que o usuario consulta
quando quer entender POR QUE o app esta mostrando determinado insight.

### Principio de design
- Linguagem sem jargao — conceitos tecnicos explicados com metaforas
- Cada pagina e independente — usuario nao precisa ler tudo em ordem
- Entrada contextual — abrir direto no grupo que o usuario esta vendo
- Nenhuma pagina bloqueia o usuario ou exige acao

### Por que funciona para os 4 objetivos
O conteudo do Coach e universal: cut, bulk, recomp e manutencao usam os mesmos
principios de volume, progressao e equilibrio muscular. A dieta muda, o treino nao.

---

## 3. ACESSO AO MODAL

### Entrada 1: Aba Mais
Novo card no grid de acoes da aba Mais (home-action-card):
```html
<div class="home-action-card" onclick="openCoachModal()" style="grid-column:1/-1;">
  <span style="font-size:20px;">🎓</span>
  <div>
    <div style="font-weight:700;font-size:13px;">Guia de Treino</div>
    <div style="font-size:11px;color:var(--text3);">Volume, progressao e equilibrio muscular</div>
  </div>
</div>
```

### Entrada 2: Botao "?" em cada mg-card (aba "Por grupo")
No header de cada card de grupo muscular, ao lado do nome:
```html
<button class="mg-coach-btn" type="button"
  onclick="openCoachModal('${grupo}');event.stopPropagation();"
  title="Guia de ${grupo}">?</button>
```

CSS do botao:
```css
.mg-coach-btn {
  width: 18px;
  height: 18px;
  border-radius: 50%;
  border: 1px solid var(--line);
  background: transparent;
  color: var(--text3);
  font-size: 11px;
  font-weight: 700;
  cursor: pointer;
  line-height: 1;
  flex-shrink: 0;
}
.mg-coach-btn:active { background: var(--surface2); }
```

---

## 4. ESTRUTURA HTML

Adicionar antes do fechamento de `</div><!-- /app -->`, apos o ultimo modal existente:

```html
<!-- COACH MODAL -->
<div class="modal-overlay" id="coachOverlay" style="z-index:330;"></div>
<div class="modal-sheet modal-full" id="coachModal" style="z-index:331;">
  <div class="modal-handle"></div>
  <div class="modal-header">
    <b id="coachModalTitle">Guia de Treino</b>
    <button class="modal-close" id="coachClose" type="button">✕</button>
  </div>
  <div class="modal-body" style="padding-top:0;">
    <!-- Tabs de navegacao entre as 5 paginas -->
    <div class="coach-nav" id="coachNav"></div>
    <!-- Conteudo da pagina ativa -->
    <div id="coachContent"></div>
    <!-- Espacamento inferior para nao cortar conteudo -->
    <div style="height:40px;"></div>
  </div>
</div>
```

---

## 5. CSS COMPLETO

Adicionar na secao de CSS do app (proxima as classes de treino):

```css
/* ═══ COACH MODAL ═══ */
.coach-nav {
  display: flex;
  gap: 6px;
  flex-wrap: nowrap;
  overflow-x: auto;
  padding: 12px 0 10px;
  margin-bottom: 4px;
  -webkit-overflow-scrolling: touch;
  scrollbar-width: none;
}
.coach-nav::-webkit-scrollbar { display: none; }

.coach-tab {
  flex-shrink: 0;
  padding: 6px 14px;
  border-radius: 20px;
  border: 1px solid var(--line);
  font-size: 12px;
  font-weight: 600;
  background: transparent;
  color: var(--text2);
  cursor: pointer;
  white-space: nowrap;
  font-family: var(--font);
}
.coach-tab.active {
  background: var(--accent);
  color: #fff;
  border-color: var(--accent);
}

.coach-section {
  margin-bottom: 24px;
}
.coach-headline {
  font-size: 15px;
  font-weight: 700;
  color: var(--text);
  margin-bottom: 10px;
}
.coach-subhead {
  font-size: 13px;
  font-weight: 700;
  color: var(--text);
  margin: 16px 0 6px;
}
.coach-body {
  font-size: 13px;
  color: var(--text2);
  line-height: 1.65;
}
.coach-body p { margin: 0 0 10px; }

.coach-quote {
  border-left: 3px solid var(--accent);
  padding: 10px 14px;
  margin: 14px 0;
  font-size: 13px;
  font-style: italic;
  color: var(--text2);
  background: var(--surface2);
  border-radius: 0 var(--radius-xs) var(--radius-xs) 0;
  line-height: 1.5;
}
.coach-quote cite {
  display: block;
  margin-top: 6px;
  font-size: 11px;
  font-style: normal;
  color: var(--text3);
  font-weight: 600;
}

.coach-table {
  width: 100%;
  border-collapse: collapse;
  font-size: 12px;
  margin: 10px 0;
}
.coach-table th {
  text-align: left;
  color: var(--text3);
  font-weight: 600;
  padding: 4px 8px;
  border-bottom: 1px solid var(--line);
  font-size: 11px;
}
.coach-table td {
  padding: 7px 8px;
  border-bottom: 1px solid rgba(255,255,255,0.05);
  vertical-align: middle;
}
.coach-table .td-bar {
  width: 80px;
}
.coach-mini-bar {
  height: 5px;
  border-radius: 3px;
  background: var(--line);
  position: relative;
}
.coach-mini-fill {
  height: 100%;
  border-radius: 3px;
  background: var(--good);
}

.coach-step {
  display: flex;
  gap: 12px;
  align-items: flex-start;
  margin-bottom: 14px;
}
.coach-step-num {
  width: 24px;
  height: 24px;
  border-radius: 50%;
  background: var(--accent);
  color: #fff;
  font-size: 12px;
  font-weight: 700;
  display: flex;
  align-items: center;
  justify-content: center;
  flex-shrink: 0;
}
.coach-step-text {
  font-size: 13px;
  color: var(--text2);
  line-height: 1.5;
  padding-top: 3px;
}

/* Selecao de grupo na pagina 5 */
.coach-grupo-grid {
  display: flex;
  flex-wrap: wrap;
  gap: 6px;
  margin-bottom: 14px;
}
.coach-grupo-btn {
  padding: 5px 12px;
  border-radius: 20px;
  border: 1px solid var(--line);
  font-size: 12px;
  font-weight: 600;
  background: transparent;
  color: var(--text2);
  cursor: pointer;
  font-family: var(--font);
}
.coach-grupo-btn.active {
  background: var(--surface2);
  border-color: var(--accent);
  color: var(--text);
}

.coach-grupo-card {
  background: var(--surface2);
  border-radius: var(--radius);
  padding: 14px;
}
.coach-grupo-header {
  font-size: 16px;
  font-weight: 700;
  color: var(--text);
  margin-bottom: 10px;
}
.coach-landmarks {
  display: flex;
  gap: 8px;
  margin-bottom: 12px;
}
.coach-lm-box {
  flex: 1;
  text-align: center;
  background: rgba(255,255,255,0.04);
  border-radius: var(--radius-xs);
  padding: 8px 4px;
}
.coach-lm-val {
  font-size: 18px;
  font-weight: 700;
  color: var(--text);
}
.coach-lm-lbl {
  font-size: 10px;
  color: var(--text3);
  font-weight: 600;
  margin-top: 2px;
}
.coach-ex-list {
  font-size: 12px;
  color: var(--text2);
  line-height: 1.8;
}
.coach-tip {
  margin-top: 10px;
  font-size: 12px;
  color: var(--accent);
  font-weight: 600;
  line-height: 1.5;
}
```

---

## 6. CONTEUDO DAS 5 PAGINAS

### Pagina 1: Fundamentos

```js
const coachPage1 = () => `
<div class="coach-section">
  <div class="coach-headline">O que e volume de treino?</div>
  <div class="coach-body">
    <p>Volume e o numero de series validas que voce faz por grupo muscular em uma semana.
    Uma serie valida e aquela feita perto da falha — quando voce teria chegado a falha
    em mais 1 a 3 repeticoes.</p>
    <p>Series muito faceis, longe da falha, nao contam como estimulo real de hipertrofia.
    O app usa <strong>series com reps > 0</strong> como proxy — e voce quem controla a intensidade.</p>
  </div>
</div>

<div class="coach-section">
  <div class="coach-headline">MEV, MAV e MRV — a faixa ideal</div>
  <div class="coach-body">
    <p><strong>MEV</strong> — Minimo Efetivo de Volume: o menor numero de series que ainda produz adaptacao.
    Abaixo disso, voce esta treinando mas nao esta crescendo (ou mantendo).</p>
    <p><strong>MAV</strong> — Maximo Adaptativo: o teto da faixa onde o ganho e maximo.</p>
    <p><strong>MRV</strong> — Maximo Recuperavel: acima disso, mais volume = menos resultado.
    O retorno diminui e a fadiga sobe exponencialmente.</p>
  </div>

  <table class="coach-table">
    <thead><tr><th>Grupo</th><th>MEV</th><th>MAV</th><th>MRV</th><th>Faixa</th></tr></thead>
    <tbody>
      ${MUSCLE_ORDER.map(g => {
        const lm = MUSCLE_LANDMARKS[g];
        const pct = Math.round((lm.mav / lm.mrv) * 100);
        return `<tr>
          <td>${g}</td>
          <td>${lm.mev}</td>
          <td>${lm.mav}</td>
          <td>${lm.mrv}</td>
          <td class="td-bar">
            <div class="coach-mini-bar">
              <div class="coach-mini-fill" style="width:${pct}%"></div>
            </div>
          </td>
        </tr>`;
      }).join("")}
    </tbody>
  </table>
</div>

<div class="coach-section">
  <div class="coach-headline">Como as abas se complementam</div>
  <div class="coach-body">
    <p><strong>"Por grupo"</strong> mostra se voce esta treinando o suficiente — o mapa do volume.</p>
    <p><strong>"Por equipamento"</strong> mostra se esse treino esta produzindo resultado — a progressao de carga.</p>
    <p>Volume OK + progressao subindo = tudo certo.<br>
    Volume OK + carga estagnada = problema de qualidade ou recuperacao.<br>
    Volume baixo + carga estagnada = provavelmente falta de estimulo.</p>
  </div>
</div>

<div class="coach-section">
  <div class="coach-headline">Exercicios compostos contam para varios grupos</div>
  <div class="coach-body">
    <p>Cada serie de um exercicio composto estimula mais de um musculo.</p>
    <p>Regra do app: motor primario = <strong>1.0 serie</strong>, sinergista = <strong>0.5 serie</strong>.</p>
    <p>Exemplos:<br>
    Supino: Peito (1.0) + Triceps e Ombros (0.5 cada)<br>
    Puxada: Costas (1.0) + Biceps (0.5)<br>
    Agachamento: Quad (1.0) + Gluteos (0.5)</p>
    <p>Por isso seus Biceps podem estar na faixa ideal mesmo sem fazer nenhuma rosca.</p>
  </div>
</div>
`;
```

### Pagina 2: Progressao e Platô

```js
const coachPage2 = () => `
<div class="coach-section">
  <div class="coach-headline">Como progredir no treino</div>
  <div class="coach-body">
    <p>Existem 3 formas de aumentar o estimulo muscular:</p>
  </div>
  <div class="coach-step">
    <div class="coach-step-num">1</div>
    <div class="coach-step-text"><strong>Mais carga</strong> — aumentar o peso usado.
    Mesmo 1kg a mais ja conta como progressao real.</div>
  </div>
  <div class="coach-step">
    <div class="coach-step-num">2</div>
    <div class="coach-step-text"><strong>Mais reps</strong> — manter a carga e fazer uma rep a mais.
    Funciona igual a aumentar o peso.</div>
  </div>
  <div class="coach-step">
    <div class="coach-step-num">3</div>
    <div class="coach-step-text"><strong>Mais volume</strong> — adicionar series ao longo das semanas.
    Referencia: +20% de volume por ciclo quando estagnado.</div>
  </div>
  <div class="coach-body" style="margin-top:8px;">
    <p>Ritmo esperado de progressao de carga:<br>
    <strong>Iniciante</strong> (menos de 3 meses): a cada 1-2 semanas<br>
    <strong>Intermediario</strong> (3-12 meses): a cada 2-3 semanas<br>
    <strong>Avancado</strong> (1 ano+): a cada 3-4 semanas ou menos frequente</p>
  </div>
</div>

<div class="coach-section">
  <div class="coach-headline">O que e um platô</div>
  <div class="coach-quote">
    Seu corpo se acostumou com o desafio atual. E como ler as mesmas 10 paginas de um livro
    varias vezes — voce nao vai aprender nada novo ate virar a pagina e aumentar o desafio.
  </div>
  <div class="coach-body">
    <p>O app detecta platô quando a carga maxima de um exercicio nao sobe por:</p>
    <p>- <strong>2 semanas</strong> para iniciantes (incomum — provavelmente falta de intensidade ou dieta)<br>
    - <strong>3 semanas</strong> para intermediarios/avancados (esperado — precisa de estrategia)</p>
  </div>
  <div class="coach-quote">
    Quanto mais treinado voce e, menos treinavel voce e.
    <cite>— Lucas Campos</cite>
  </div>
</div>

<div class="coach-section">
  <div class="coach-headline">Como resolver um platô</div>
  <div class="coach-subhead">Para iniciantes</div>
  <div class="coach-body">
    <p>Questione a intensidade: as series estao chegando perto da falha?
    Questione a dieta: proteina e calorias suficientes para o objetivo?</p>
  </div>
  <div class="coach-subhead">Para intermediarios/avancados</div>
  <div class="coach-body">
    <p>Opcao 1: Mudar a faixa de reps (ex: de 10-12 para 6-8 ou 15-20).<br>
    Opcao 2: Trocar o exercicio por uma variacao.<br>
    Opcao 3: Fazer um ciclo de volume (ver pagina Deload).</p>
  </div>
</div>

<div class="coach-section">
  <div class="coach-headline">Faixas de reps — o que importa</div>
  <div class="coach-body">
    <p>Para hipertrofia, qualquer faixa entre <strong>5 e 30 reps</strong> funciona — se a serie
    for feita perto da falha. O numero de reps e menos importante do que a intensidade.</p>
    <p>Variar a faixa e recomendado para:<br>
    - Conforto articular (cargas altas por muito tempo podem sobrecarregar tendoes)<br>
    - Novos estimulos para musculos acostumados ao mesmo padrao</p>
  </div>
</div>
`;
```

### Pagina 3: Volume Cycling

```js
const coachPage3 = () => `
<div class="coach-section">
  <div class="coach-headline">Por que reduzir o volume as vezes?</div>
  <div class="coach-quote">
    Pense no seu treino como uma viagem de carro: de vez em quando precisamos parar no posto,
    nao para abandonar a viagem, mas para garantir que o motor nao quebre e possamos
    chegar mais longe.
  </div>
  <div class="coach-body">
    <p>Apos semanas de volume alto, o musculo perde sensibilidade ao estimulo.
    Reduzir temporariamente o volume <strong>ressensibiliza</strong> o musculo —
    ele responde melhor quando o volume voltar.</p>
    <p>Isso nao e fraqueza. E estrategia.</p>
  </div>
</div>

<div class="coach-section">
  <div class="coach-headline">Volume Cycling vs Deload tradicional</div>
  <div class="coach-body">
    <p>Lucas Campos NAO usa o conceito de "deload" como repouso ou reducao de carga.
    O conceito correto e <strong>Volume Cycling</strong>:</p>
  </div>
  <div class="coach-step">
    <div class="coach-step-num">✓</div>
    <div class="coach-step-text"><strong>Reduzir drasticamente o VOLUME</strong> — de 12-15 series
    para apenas 3-4 series por grupo na semana</div>
  </div>
  <div class="coach-step">
    <div class="coach-step-num">✓</div>
    <div class="coach-step-text"><strong>MANTER a intensidade (carga)</strong> — o peso permanece o mesmo.
    Nao e para ficar mais facil, e para fazer menos</div>
  </div>
  <div class="coach-step">
    <div class="coach-step-num">✓</div>
    <div class="coach-step-text"><strong>Duracao de 1 a 2 semanas</strong> — tempo suficiente para
    recuperar sem perder massa (com apenas 20% do volume previo, o musculo mantem)</div>
  </div>
  <div class="coach-body" style="margin-top:8px;">
    <p>Depois do ciclo: retome o volume anterior. Voce vai notar melhor recuperacao
    e, frequentemente, um salto de desempenho.</p>
  </div>
</div>

<div class="coach-section">
  <div class="coach-headline">Quando o app vai sugerir</div>
  <div class="coach-body">
    <p>O app sinaliza necessidade de Volume Cycling em dois casos:</p>
    <p><strong>1. Volume cronicamente alto:</strong> grupo acima do MAV por 4 ou mais semanas seguidas.<br>
    <strong>2. Queda de forca cruzada:</strong> carga maxima caindo em 2 ou mais exercicios
    do mesmo grupo na mesma semana — sinal de fadiga acumulada sistemica.</p>
    <p>Os chips de insight aparecem no rodape do card do grupo afetado.</p>
  </div>
</div>

<div class="coach-section">
  <div class="coach-headline">O que esperar depois</div>
  <div class="coach-body">
    <p>A ressensibilizacao muscular significa que seu corpo vai responder melhor ao
    volume quando ele voltar. Muitos usuarios relatam uma das melhores semanas de
    treino logo apos um ciclo de volume bem feito.</p>
  </div>
</div>
`;
```

### Pagina 4: Equilibrio Muscular

```js
const coachPage4 = () => `
<div class="coach-section">
  <div class="coach-headline">Por que treinar todos os grupos</div>
  <div class="coach-quote">
    Treinar o corpo de forma desigual e como construir uma casa com paredes fortes e
    um teto fraco. Para ter um shape bonito e saudavel, precisamos distribuir o trabalho
    de forma que nenhuma parte fique sobrecarregada ou esquecida.
  </div>
  <div class="coach-body">
    <p>Musculos antagonistas se limitam mutuamente. Peito muito mais forte que costas
    cria desequilibrio postural e pode limitar o proprio desenvolvimento do peito.
    Quad muito mais forte que posterior aumenta risco de lesao no joelho.</p>
  </div>
</div>

<div class="coach-section">
  <div class="coach-headline">Pares antagonistas</div>
  <div class="coach-body">
    <p>O app monitora esses 3 pares para detectar desequilibrio:</p>
  </div>
  <table class="coach-table">
    <thead><tr><th>Par</th><th>Grupo A</th><th>Grupo B</th></tr></thead>
    <tbody>
      <tr><td>Empurrar/Puxar</td><td>🏋️ Peito</td><td>🦅 Costas</td></tr>
      <tr><td>Frente/Tras da coxa</td><td>🦵 Quad</td><td>🦵 Posterior</td></tr>
      <tr><td>Braco</td><td>💪 Biceps</td><td>💪 Triceps</td></tr>
    </tbody>
  </table>
  <div class="coach-body">
    <p>O alerta dispara quando: um grupo tem mais de 2.5x o volume do antagonista
    <strong>E</strong> o menor esta abaixo do MEV.</p>
    <p>Se ambos estao acima do MEV, nao e desequilibrio clinico — e escolha do usuario.</p>
  </div>
</div>

<div class="coach-section">
  <div class="coach-headline">A Regra da Prioridade</div>
  <div class="coach-body">
    <p>Lucas Campos nao exige uma proporcao fixa (como 1:1) entre grupos antagonistas.
    Ele defende a <strong>Regra da Prioridade</strong>: o grupo que voce quer desenvolver mais
    pode ter mais volume.</p>
    <p>A restricao e simples: o grupo antagonista nunca pode ficar abaixo do MEV.
    Isso e o minimo para manter saude articular e equilibrio postural.</p>
    <p>O app nunca vai dizer "treine menos X". Sempre vai dizer "Y esta abaixo do minimo".</p>
  </div>
</div>

<div class="coach-section">
  <div class="coach-headline">Funciona para todos os objetivos</div>
  <div class="coach-body">
    <p>Cut, bulk, recomp ou manutencao — o equilibrio muscular e sempre o mesmo.
    O que muda entre os objetivos e a dieta, nao a distribuicao do treino.</p>
    <p>O monitoramento de volume e universal porque o estimulo muscular funciona
    pelos mesmos mecanismos independente do objetivo calorico.</p>
  </div>
</div>
`;
```

### Pagina 5: Guia por Grupo

```js
// Dados de cada grupo para a pagina 5
const COACH_GRUPO_DATA = {
  "🏋️ Peito": {
    freq: "2x/semana (ideal)",
    exercicios: ["Supino reto (barra ou halter)", "Supino inclinado", "Crossover (cabo)", "Peck deck (voador)", "Flexao de braco"],
    melhorRetorno: "Supino reto — maior ativacao e maior sobrecarga progressiva",
    dica: "Volume indireto de Triceps e Ombros ja conta via supino. Some tudo antes de adicionar isolados.",
    obs: "Supino inclinado enfatiza a porcao superior. Crucifixo e crossover complementam com ativacao em alongamento."
  },
  "🦅 Costas": {
    freq: "2x/semana (ideal)",
    exercicios: ["Puxada frontal", "Barra fixa", "Remada curvada (barra)", "Remada unilateral (halter)", "Remada baixa (cabo)"],
    melhorRetorno: "Puxada frontal / barra fixa — maior amplitude de movimento",
    dica: "Cada serie de costas traz 0.5 serie de Biceps de brinde. Verifique o total antes de adicionar roscas.",
    obs: "Remadas enfatizam porcao media da costas. Puxadas enfatizam latissimo. Ideal ter um de cada."
  },
  "🦵 Quad": {
    freq: "1-2x/semana",
    exercicios: ["Agachamento livre (barra)", "Leg press 45", "Cadeira extensora", "Hack squat", "Agachamento bulgaro"],
    melhorRetorno: "Agachamento livre — maior ativacao e estimulo hormonal",
    dica: "Pernas no inicio do treino — qualidade das series cai muito com fadiga acumulada.",
    obs: "Cadeira extensora e util para finalizar com foco em quad, especialmente se o agachamento nao for possivel."
  },
  "🦵 Posterior": {
    freq: "1-2x/semana",
    exercicios: ["Mesa flexora", "Cadeira flexora", "Stiff (barra/halter)", "Stiff romeno / RDL"],
    melhorRetorno: "Stiff — maior carga possivel e ativacao em alongamento",
    dica: "O grupo mais negligenciado das pernas. 1 mesa flexora + 1 stiff por semana ja e suficiente para sair do cinza.",
    obs: "Posterior recupera mais rapido que quad — pode treinar em dias consecutivos se necessario."
  },
  "🍑 Glúteos": {
    freq: "2-3x/semana (aguenta volume alto)",
    exercicios: ["Hip thrust (barra ou maquina)", "Stiff romeno / RDL", "Elevacao pelvica", "Gluteo no cabo (kickback)"],
    melhorRetorno: "Hip thrust — maior carga e ativacao direta de gluteo maximo",
    dica: "MEV de 15 series — mais alto que qualquer outro grupo. Volume baixo simplesmente nao funciona para gluteos.",
    obs: "Gluteos sao o maior grupo muscular do corpo. Recuperam rapido e respondem bem a frequencia alta (2-3x/sem)."
  },
  "💪 Ombros": {
    freq: "2x/semana",
    exercicios: ["Elevacao lateral (halter ou cabo)", "Desenvolvimento (halter ou barra)", "Face pull (corda)", "Crucifixo inverso"],
    melhorRetorno: "Elevacao lateral — exercicio mais eficiente por serie para o deltoides medial",
    dica: "Desenvolvimento ja traz volume indireto de Triceps (0.5x). Face pull trabalha posterior e manguito rotador.",
    obs: "Deltoides anterior ja recebe muito volume via supinos. Foque em lateral e posterior para equilibrio."
  },
  "💪 Bíceps": {
    freq: "2x/semana",
    exercicios: ["Rosca direta (barra)", "Rosca alternada (halter)", "Rosca martelo", "Rosca Scott", "Rosca no cabo"],
    melhorRetorno: "Rosca direta com barra — maior carga absoluta",
    dica: "Muito volume ja vem de costas (0.5x por serie de puxada/remada). Some o indireto antes de decidir quantas roscas fazer.",
    obs: "Rosca martelo ativa mais braquiorradial. Rosca no cabo mantem tensao constante. Boa combinacao."
  },
  "💪 Tríceps": {
    freq: "2x/semana",
    exercicios: ["Triceps pulley (corda ou barra V)", "Triceps testa", "Triceps frances (halter)", "Paralelas"],
    melhorRetorno: "Triceps testa / frances — maior ativacao em alongamento",
    dica: "Muito volume ja vem do peito e ombros (0.5x por serie de supino/desenvolvimento). Some tudo antes de adicionar mais isolados.",
    obs: "Pulley e bom para volume alto com baixo risco articular. Exercicios em alongamento (frances, testa) geram mais hipertrofia por serie."
  },
  "🧱 Core": {
    freq: "2-3x/semana (recupera rapido)",
    exercicios: ["Prancha (isometria)", "Abdominal crunch", "Roda abdominal", "Rotacao russa", "Abdominal infra"],
    melhorRetorno: "Roda abdominal — maior amplitude e carga relativa",
    dica: "Core recupera rapido — pode treinar ao final de qualquer sessao sem prejudicar o treino principal.",
    obs: "MEV de 4 series e baixo porque o core e recrutado indiretamente em agachamentos, terra e remadas."
  }
};

const coachPage5 = (grupoInicial) => `
<div class="coach-section">
  <div class="coach-headline">Guia por grupo muscular</div>
  <div class="coach-body" style="margin-bottom:10px;">
    Selecione um grupo para ver MEV, MRV, exercicios recomendados e dicas especificas.
  </div>
  <div class="coach-grupo-grid">
    ${MUSCLE_ORDER.map(g => `
      <button class="coach-grupo-btn ${g === (grupoInicial || MUSCLE_ORDER[0]) ? 'active' : ''}"
        type="button" onclick="renderCoachGrupo('${g}', this)">${g}</button>
    `).join("")}
  </div>
  <div id="coachGrupoCard"></div>
</div>
`;
```

---

## 7. JAVASCRIPT COMPLETO

### Funcao renderCoachGrupo(grupo, btnEl)

```js
const renderCoachGrupo = (grupo, btnEl) => {
  // Update botoes ativos
  document.querySelectorAll(".coach-grupo-btn").forEach(b => b.classList.remove("active"));
  if (btnEl) btnEl.classList.add("active");

  const data = COACH_GRUPO_DATA[grupo];
  const lm = MUSCLE_LANDMARKS[grupo];
  if (!data || !lm) return;

  const card = document.getElementById("coachGrupoCard");
  if (!card) return;

  card.innerHTML = `
    <div class="coach-grupo-card">
      <div class="coach-grupo-header">${grupo}</div>
      <div class="coach-landmarks">
        <div class="coach-lm-box">
          <div class="coach-lm-val">${lm.mev}</div>
          <div class="coach-lm-lbl">MEV</div>
        </div>
        <div class="coach-lm-box">
          <div class="coach-lm-val">${lm.mav}</div>
          <div class="coach-lm-lbl">MAV</div>
        </div>
        <div class="coach-lm-box">
          <div class="coach-lm-val">${lm.mrv}</div>
          <div class="coach-lm-lbl">MRV</div>
        </div>
      </div>
      <div class="coach-subhead" style="margin-top:0;">Frequencia ideal</div>
      <div class="coach-body" style="margin-bottom:10px;">${data.freq}</div>
      <div class="coach-subhead">Exercicios recomendados</div>
      <div class="coach-ex-list">${data.exercicios.map(e => `• ${e}`).join("<br>")}</div>
      <div class="coach-subhead">Maior retorno por serie</div>
      <div class="coach-body" style="margin-bottom:0;">${data.melhorRetorno}</div>
      <div class="coach-tip">💡 ${data.dica}</div>
      ${data.obs ? `<div class="coach-body" style="margin-top:10px;font-size:12px;color:var(--text3);">${data.obs}</div>` : ""}
    </div>
  `;
};
```

### Funcao renderCoachPage(pageId, grupoInicial)

```js
const COACH_PAGES = [
  { id: "fundamentos",  label: "Fundamentos",  fn: () => coachPage1() },
  { id: "progressao",   label: "Progressao",   fn: () => coachPage2() },
  { id: "cycling",      label: "Deload",        fn: () => coachPage3() },
  { id: "equilibrio",   label: "Equilibrio",   fn: () => coachPage4() },
  { id: "por-grupo",    label: "Por grupo",    fn: (g) => coachPage5(g) },
];

const renderCoachPage = (pageId, grupoInicial) => {
  const page = COACH_PAGES.find(p => p.id === pageId);
  if (!page) return;

  // Atualizar tabs
  document.querySelectorAll(".coach-tab").forEach(t => {
    t.classList.toggle("active", t.dataset.page === pageId);
  });

  // Renderizar conteudo
  const content = document.getElementById("coachContent");
  content.innerHTML = page.fn(grupoInicial);

  // Se for pagina de grupo, renderizar o primeiro grupo
  if (pageId === "por-grupo") {
    const alvo = grupoInicial || MUSCLE_ORDER[0];
    const btn = content.querySelector(`.coach-grupo-btn[onclick*="'${alvo}'"]`);
    renderCoachGrupo(alvo, btn);
  }
};
```

### Funcao renderCoachNav()

```js
const renderCoachNav = () => {
  const nav = document.getElementById("coachNav");
  nav.innerHTML = COACH_PAGES.map(p => `
    <button class="coach-tab" type="button" data-page="${p.id}"
      onclick="renderCoachPage('${p.id}')">${p.label}</button>
  `).join("");
};
```

### openCoachModal / closeCoachModal

```js
const openCoachModal = (grupoInicial) => {
  renderCoachNav();
  // Se veio com grupo especifico, abrir na pagina "Por grupo"
  if (grupoInicial && COACH_GRUPO_DATA[grupoInicial]) {
    renderCoachPage("por-grupo", grupoInicial);
    document.getElementById("coachModalTitle").textContent = grupoInicial;
  } else {
    renderCoachPage("fundamentos");
    document.getElementById("coachModalTitle").textContent = "Guia de Treino";
  }
  document.getElementById("coachOverlay").classList.add("open");
  document.getElementById("coachModal").classList.add("open");
};

const closeCoachModal = () => {
  document.getElementById("coachOverlay").classList.remove("open");
  document.getElementById("coachModal").classList.remove("open");
  document.getElementById("coachModalTitle").textContent = "Guia de Treino";
};

// Listeners (adicionar no bloco de inicializacao)
document.getElementById("coachClose").addEventListener("click", closeCoachModal);
document.getElementById("coachOverlay").addEventListener("click", closeCoachModal);
```

### Adicionar botao "?" nos mg-cards (dentro de renderMuscleVolume)

No HTML de cada card, no `.mg-header`, adicionar o botao apos o nome:
```js
// Dentro do loop de MUSCLE_ORDER em renderMuscleVolume():
html += `
  <div class="mg-card ${cardClass}" data-grupo="${grupo}">
    <div class="mg-header">
      <span class="mg-name">${grupo}</span>
      <div style="display:flex;align-items:center;gap:8px;">
        <button class="mg-coach-btn" type="button"
          onclick="openCoachModal('${grupo}');event.stopPropagation();"
          title="Guia de ${grupo}">?</button>
        <span class="mg-sets">${totalRounded} sets</span>
      </div>
    </div>
    ...resto do card...
  </div>`;
```

---

## 8. PLANO DE IMPLEMENTACAO (3 partes)

### Parte 1: HTML + CSS
- Adicionar `#coachOverlay` e `#coachModal` no HTML (antes de `</div><!-- /app -->`)
- Adicionar todo o bloco CSS `.coach-*` e `.mg-coach-btn` na secao de estilos
- Verificar z-index: overlay=330, modal=331

### Parte 2: Conteudo e funcoes JS
Adicionar na ordem, apos as funcoes de v2.9.0:
1. `COACH_GRUPO_DATA` — objeto com dados de cada grupo
2. `coachPage1()` ... `coachPage5()` — funcoes geradoras de HTML
3. `COACH_PAGES` — array de paginas
4. `renderCoachGrupo(grupo, btnEl)`
5. `renderCoachPage(pageId, grupoInicial)`
6. `renderCoachNav()`
7. `openCoachModal(grupoInicial)` / `closeCoachModal()`

### Parte 3: Integracao
- Listeners: `#coachClose` e `#coachOverlay` chamam `closeCoachModal()`
- Card na aba Mais: novo `home-action-card` abrindo `openCoachModal()`
- Botao "?" em cada `.mg-card`: adicionar no HTML gerado por `renderMuscleVolume()`
- sw.js: bump de `kcalix-v13` para `kcalix-v14`

---

## 9. CHECKLIST DE VALIDACAO

### HTML + CSS
- [ ] Modal abre e fecha sem travar scroll do app
- [ ] Overlay clicavel fecha o modal
- [ ] Nav tabs scrollam horizontalmente em telas estreitas (375px) sem quebrar layout
- [ ] Conteudo de todas as paginas renderiza sem overflow horizontal
- [ ] Tabela de landmarks renderiza corretamente em 375px
- [ ] Card de grupo (pagina 5) exibe MEV/MAV/MRV corretamente

### Funcionalidade
- [ ] `openCoachModal()` sem argumento abre em "Fundamentos"
- [ ] `openCoachModal("Gluteos")` abre em "Por grupo" com Gluteos pre-selecionado
- [ ] Botoes das tabs navegam entre paginas corretamente
- [ ] Selecao de grupo na pagina 5 atualiza o card corretamente
- [ ] Botao "?" em cada mg-card abre o modal no grupo correto
- [ ] Botao na aba Mais abre o modal normalmente
- [ ] Titulo do modal muda para o nome do grupo quando aberto por contexto

### Conteudo
- [ ] Pagina 1: tabela de MEV/MAV/MRV gerada dinamicamente (nao hardcoded)
- [ ] Pagina 5: todos os 9 grupos tem dados em COACH_GRUPO_DATA
- [ ] Metaforas do Lucas presentes nas paginas corretas (plateau, deload, desequilibrio)
- [ ] Nenhuma pagina menciona "treine menos X" — sempre "Y abaixo do minimo"

### Compatibilidade
- [ ] Abas "Por treino" e "Por equipamento" continuam funcionando
- [ ] Insights automaticos (v2.9.0) continuam aparecendo nos cards
- [ ] sw.js bumped para kcalix-v14
- [ ] Sem erros de console em nenhuma pagina

---

## 10. METAFORAS DO LUCAS CAMPOS (usar exatamente estas, nas paginas indicadas)

```
PLATEAU (pagina 2 — Progressao):
"Seu corpo se acostumou com o desafio atual. E como ler as mesmas
10 paginas de um livro varias vezes — voce nao vai aprender nada
novo ate virar a pagina e aumentar o desafio."

DELOAD / VOLUME CYCLING (pagina 3 — Deload):
"Pense no seu treino como uma viagem de carro: de vez em quando
precisamos parar no posto, nao para abandonar a viagem, mas para
garantir que o motor nao quebre e possamos chegar mais longe."

DESEQUILIBRIO (pagina 4 — Equilibrio):
"Treinar o corpo de forma desigual e como construir uma casa com
paredes fortes e um teto fraco. Para ter um shape bonito e saudavel,
precisamos distribuir o trabalho de forma que nenhuma parte fique
sobrecarregada ou esquecida."

NIVEL DE TREINAMENTO (pagina 2 — Progressao):
"Quanto mais treinado voce e, menos treinavel voce e."
— Lucas Campos
```

---

## 11. FORA DO ESCOPO v3.0.0 (fases futuras)

- Sistema de prioridades: usuario marca grupos prioritarios → app ajusta thresholds
- MEV/MAV configuravel pelo usuario
- Sugestao automatica de treino para fechar deficit
- Grafico radar de distribuicao muscular
- Correlacao volume de treino x progresso corporal
- Busca dentro do Coach Modal
- Coach contextual inline nos insights (sem abrir modal separado)

---

## 12. RESUMO EXECUTIVO PARA NOVA SESSAO

### Pre-requisitos
- v2.8.0 deployada: calcMuscleVolume, MUSCLE_LANDMARKS, MUSCLE_ORDER, aba "Por grupo"
- v2.9.0 deployada: buildInsightsByGroup, mg-chip-row, mgChipToggle, getUserLevel

### O que v3.0.0 entrega
Modal educativo `#coachModal` (z-index 331 / overlay 330) com 5 paginas:
1. Fundamentos — series validas, MEV/MAV/MRV, tabela por grupo, como as abas se complementam
2. Progressao e Platô — 3 formas de progredir, criterios de platô por nivel, faixas de reps
3. Deload (Volume Cycling) — conceito do Lucas, como fazer, quando o app vai sugerir
4. Equilibrio Muscular — pares antagonistas, Regra da Prioridade, por que funciona para todos os objetivos
5. Por Grupo — guia especifico: MEV/MAV/MRV, exercicios, maior retorno por serie, dica do Lucas

### Acessos
- Botao "Guia de Treino" na aba Mais
- Botao "?" em cada card de grupo na aba "Por grupo" → abre direto no grupo

### Implementacao
3 partes: HTML+CSS → funcoes JS (COACH_GRUPO_DATA + paginas + render) → integracao (listeners + botoes + sw.js)
