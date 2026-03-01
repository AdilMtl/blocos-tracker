# Changelog

Todas as mudanças notáveis do Kcal.ix são documentadas aqui.
Formato baseado em [Keep a Changelog](https://keepachangelog.com/pt-BR/1.0.0/).

---

## [v1.8.3] — 2026-03-01

### Adicionado
- [feat] **Banner de instalação PWA** fixo acima da bottom-nav:
  - **iOS Safari**: instrução manual "Compartilhar → Adicionar à Tela de Início"
  - **Android Chrome**: captura `beforeinstallprompt` e exibe botão "Instalar" com prompt nativo
  - Dismiss via ✕ salvo em `blocos_tracker_pwa_dismissed` (não reaparece)
  - Esconde automaticamente via evento `appinstalled`
  - Touch targets dos botões ≥ 40px

### Corrigido
- [fix] **manifest.json**: `scope` e `start_url` agora absolutos (`/blocos-tracker/`) — resolve abertura em modo browser em vez de standalone no iOS
- [fix] **manifest.json**: `"purpose": "any maskable"` separado em duas entradas distintas — compatibilidade com validadores iOS/Safari
- [fix] **`apple-mobile-web-app-title`**: "Blocos" → "Kcal.ix"
- [fix] **Diet auto-check**: hábito "dieta" agora só é marcado automaticamente se jantar estiver preenchido (indica fim do dia) — antes disparava pela manhã com qualquer valor ≤ meta
- [fix] **Export JSON IA**: `totalKcal` em `nutrition.days` usa `settings.kcalPerBlock` (idêntico ao app) em vez de `blocks.pG × 4` — elimina divergência em configs personalizadas
- [fix] **`btnSave`**: agora persiste `foodLog` explicitamente junto com `data` — eliminava risco de perda do log de alimentos após reload sem autosave

### Notas
- **Para testar**: validar banner iOS e fluxo de instalação em device real (iPhone)
- localStorage: nenhuma chave alterada; nova chave: `blocos_tracker_pwa_dismissed`

---

## [v1.8.0] — 2026-03-01

### Adicionado
- [feat] **Timer de Pausa** no viewTreino — accordion "⏱ Timer de Pausa" com:
  - 5 presets configuráveis (padrão: 30s · 1:00 · 1:30 · 2:00 · 3:00) salvos em `blocos_tracker_timer_presets_v1`
  - Um toque no preset seleciona e inicia imediatamente
  - Long press (600ms) no preset abre edição inline (prompt)
  - Botões Stop e Reset
  - Display grande MM:SS com estado visual (branco → verde rodando → roxo finalizado)
- [feat] **Notificação nativa** ao fim da pausa — funciona com o app minimizado (Android Chrome e iOS PWA instalada ≥ 16.4)
  - Permissão solicitada uma vez no primeiro uso; timer funciona normalmente se negada
  - Clicar na notificação foca/abre o app via `notificationclick` no Service Worker
- [feat] **Cronômetro** (conta pra cima) — aba secundária dentro do mesmo accordion, start/pause/reset
- [feat] **sw.js**: handler `notificationclick` para focar janela existente ou abrir nova

### Técnico
- `CACHE_NAME`: `kcalix-v1` → `kcalix-v2` (bump obrigatório pela mudança no sw.js)
- `setTimeout` usa `Date.now()` como referência (sem drift acumulado em background)
- Nova chave localStorage: `blocos_tracker_timer_presets_v1` (array de inteiros em segundos)

### Notas
- **iOS sem PWA instalada**: notificações não funcionam no Safari sem instalar o app na tela inicial — limitação da plataforma Apple.
- localStorage: nenhuma chave existente alterada; dados de treino intactos.

---

## [v1.7.0] — 2026-03-01

### Adicionado
- [feat] **Energy Analytics Dashboard** na Home — painel de análise energética diária e semanal:
  - `getEnergyForDate(date)`: helper puro que retorna `{consumed, bmr, tdee, exercise, total, balance}` para qualquer dia.
  - Card **"⚡ Energia Hoje"**: 4 KPIs inline (kcal consumido / gasto basal / treino / saldo) com cor semântica — verde para déficit, roxo para superávit.
  - Barra de progresso consumido vs meta kcal.
  - **Gráfico semanal Seg–Dom**: barras duplas sobrepostas (gasto atrás em surface3, consumido na frente em accent) + linha meta tracejada. Dia atual com borda destacada; dias futuros sem barra; dias sem dados com opacidade reduzida.
  - **Projeção kg/semana**: média de saldo dos últimos 7 dias × 7 / 7700 — exibida se ≥ 3 dias com dados (📉 déficit / 📈 superávit).
- [feat] **Saudação contextual vs histórico** — ao visualizar hoje: "Bom dia/tarde/noite 👋"; ao visualizar dia passado: "📅 Histórico". Dados e cards atualizam em tempo real ao mudar a data.

### Melhorado
- [improve] **Home reativa aos 3 caminhos de mudança de data** — `renderHomeDashboard()` agora é chamado explicitamente em: (1) `datePick.change`, (2) `shiftDate()` — botões ‹ ›, (3) `btnGoToday` — botão "hoje". Antes era necessário clicar na aba Home para atualizar.

### Corrigido
- [fix] `barWrap.innerHTML` não era resetado no path `hasBmr=true` de `renderEnergyCard()` — a cada re-render uma nova barra era acumulada sobre a anterior.
- [fix] `.week-bar-wrap` sem altura efetiva (colapsava para `min-height:4px` por `flex:1` sem pai esticado) — corrigido com altura explícita via `style="height:${chartH}px"` inline.

### Notas
- **Requer JP7 configurado** (Mais → Calculadora) para exibir barras de gasto basal/total no gráfico. Sem JP7, o card mostra apenas consumido + mensagem instrutiva. Isso é intencional — sem BMR não é possível calcular gasto real.
- **Ideia futura**: tela de primeiro acesso / onboarding quando não há nenhum dado registrado — guiar o usuário a configurar JP7, metas e primeiro log antes de exibir o dashboard.
- localStorage: **nenhuma mudança de schema** — dados existentes totalmente compatíveis.

---

## [v1.6.0] — 2026-02-28

### Adicionado
- [feat] **Auto-check de hábitos ao salvar dados** — ao salvar Diário (hábito "log" e "dieta"), Treino/Cardio ("treino" e "cardio") e Medidas ("medidas"), o hábito correspondente é marcado automaticamente para o dia de hoje, com toast de confirmação. Guard `currentDate() === todayISO()` garante que só afeta o dia atual.

### Melhorado
- [improve] **FAB (+) removido** — era redundante com o grid 2×2 da Home e causava confusão sobre sua função.

### Notas
- localStorage: nenhuma mudança de schema.

---

## [v1.5.0] — 2026-02-28

### Identidade
- [feat] **App renomeado: Blocos Tracker → Kcal.ix** — `<title>`, `<h1>`, `manifest.json` (name/short_name), prompt do sistema IA e nome do arquivo de export JSON. Storage keys preservados intactos (sem perda de dados).

### Adicionado
- [feat] **Home dashboard** — nova aba padrão com saudação contextual (Bom dia/tarde/noite), data formatada, card de progresso calórico (barra kcal consumida vs meta) e 3 mini-barras de macros (Proteína / Carbo / Gordura em gramas vs meta).
- [feat] **Grid 2×2 de ações rápidas** na Home — blocos quadrados tocáveis: 📊 Diário · 🏋️ Treino · 📏 Corpo · 🤖 IA Export. Toque no card de progresso navega direto ao Diário.
- [feat] **FAB (+)** — botão flutuante fixo (placeholder visual; Fase 2 implementa bottom sheet de quick-add).
- [feat] **Abertura inteligente via sessionStorage** — app abre na Home na primeira abertura; mantém a aba ativa ao trocar de app e voltar (sessionStorage limpa ao fechar a aba do browser).

### Melhorado
- [improve] **Bottom-nav: 6 → 5 abas** — Home · Diário · Treino · Corpo · Mais. Thumb zone otimizada; `grid-template-columns: repeat(5, 1fr)`.
- [improve] **Aba "Mais"** unifica Ajustes + Calculadora JP7 (antes em abas separadas) num único accordion, limpando a nav principal.
- [improve] **Alimentos integrado no Diário** — seção "🍽️ Alimentos" como accordion colapsável no fim da aba Diário (busca, log do dia, alimento personalizado). Não ocupa mais uma aba dedicada na nav.
- [improve] **Habit Tracker movido para a Home** — removido do topo do Diário; `renderHabitTracker()` continua apontando para `#habitTracker` sem mudança de lógica.

### Corrigido
- [fix] `grid-template-columns: repeat(6→5, 1fr)` no `.bottom-nav-inner` — com 5 tabs, o 6º slot gerava coluna vazia e tabs menores.

### Infraestrutura
- Service Worker: `blocos-v5` → `kcalix-v1` (limpeza de cache antigo; necessária por causa do rename).
- `PLAN.md` adicionado ao repositório — plano de reestruturação com todas as etapas [x] marcadas.

### Notas
- localStorage: **nenhuma mudança de schema** — todos os dados do usuário compatíveis com versões anteriores.
- ⚠️ Bug pré-existente registrado (não introduzido nesta versão): `select#tmplEditCardioType` tem `font-size: 13px` → pode causar zoom automático no iOS ao focar. Correção planejada.
- Próxima fase (Fase 2): FAB com bottom sheet de quick-add (Alimento / Treino / Medição rápida).

---

## [v1.4.0] — 2026-02-27

### Adicionado
- [feat] **Habit Tracker semanal** no topo da aba Tracker — grid Seg–Dom com 5 hábitos (🥗 Dieta, 🍽️ Log, 🏋️ Treino, 🏃 Cardio, 📏 Medidas). Círculos neon por hábito com glow effect ao marcar. Dias futuros bloqueados. Score do dia (ex: 3/5) sempre visível no cabeçalho.
- [feat] Habit Tracker **retrátil** — toggle com animação suave (`max-height`). Estado aberto/fechado persistido em `localStorage`. Colapsado exibe score dots coloridos + contagem ao lado do título.
- [feat] **Botão "hoje"** no header — pill roxo minimalista que aparece (fade) ao navegar para outro dia e some ao voltar para hoje. Redefine data e sincroniza todas as views.

### Infraestrutura
- Service Worker: `blocos-v4` → `blocos-v5`.
- Novo localStorage key: `blocos_tracker_habits_v1` — schema `{ "YYYY-MM-DD": { "dieta": true, "log": false, ... } }`. Não toca em nenhum dado existente.

### Notas
- Ideia futura: auto-check de Treino e Medidas quando o usuário salva dados nessas seções (hoje é 100% manual).

---

## [v1.3.0] — 2026-02-27

### Adicionado
- [feat] Export para Coach IA: card "🤖 Exportar para IA" na aba Macros com dois botões — **⬇️ Baixar JSON** (últimos 60 dias de nutrição, treino, medições, perfil e log detalhado de alimentos) e **📋 Copiar prompt do sistema** (pronto para Custom GPT ou Gemini Gem).
- [feat] `nutrition.foodLog` no JSON exportado: log detalhado de cada alimento selecionado por dia/refeição (nome, qty, gramas reais P/C/G, blocos, kcal, timestamp).

### Melhorado
- [improve] FOOD_DB revisado com base em TACO & Atwater: 104 → 109 alimentos. Categoria renomeada para "Pães, Cereais & Raízes" com adição de batata doce, mandioca e macarrão. Novas carnes: tilápia, salmão, lombo suíno. Novos legumes: abóbora, couve-flor, espinafre. Fast-food: +sushi salmão, temaki, batata frita, cachorro-quente. Doces: +doce de leite, goiabada, paçoca, gelatina. Valores nutricionais corrigidos em todas as categorias.
- [improve] Prompt do sistema IA reescrito: estrutura correta do JSON exportado, mapeamento de mealId para nomes de refeições, distinção explícita entre valores em blocos (nutrition.days) e em gramas (foodLog.entries).

### Corrigido
- [fix] Export JSON não disparava download: `loadJSON()` chamado sem fallback gerava `SyntaxError` silencioso — substituído por variáveis IIFE já carregadas.
- [fix] `measure.entries` → `measure.days` (medições exportadas saíam sempre vazias).
- [fix] 5 campos com nomes errados no export: `settings.meals` → `MEALS`; `day.meals[i]` → `day.meals[m.id]`; `settings.pBlockG/goalP` → `settings.blocks.pG / settings.goals.pG`; `calc.weight` → `calc.weightKg`.
- [fix] Aderência calculada com unidades mistas (blocos vs gramas) — normalizado para blocos em ambos os lados.

### Infraestrutura
- Service Worker: `blocos-v3` → `blocos-v4` (limpeza de cache antigo no celular).

### Notas
- localStorage: nenhuma mudança de schema nos dados existentes. O export é somente leitura.
- Próximo passo planejado: integração direta de IA no app (API key do usuário, sem necessidade de exportar manualmente).

---

## [v1.2.0] — 2026-02-27

### Adicionado
- [feat] Medição: modal "Ver evolução 📈" com gráfico SVG nativo — filtrável por Peso (kg), Cintura (cm) e BF%. Tooltip por ponto, resumo com Mínimo / Máximo / Atual / Variação (▲▼).
- [feat] Treino: grupo 🍑 Glúteos com 7 exercícios (Hip thrust barra/máquina, Máquina de glúteos, Glúteo no cabo, Elevação pélvica, Stiff romeno/RDL, Agachamento sumô). Total: 63 → 70 exercícios.

### Melhorado
- [improve] Treino: proteção ao trocar de template — confirm() se houver séries preenchidas, evitando perda acidental de dados.
- [improve] Treino: chip de volume por exercício (roxo, ex: "2.4k vol") exibido ao lado do badge de carga máxima no header de cada exercício.
- [improve] Treino: toggle Carga máx / Volume no gráfico de barras do modal de progressão 📊 — barra roxa para volume, azul para carga.
- [improve] Treino: cálculo de kcal de musculação escala por reps × carga com piso 5 / teto 14 kcal por série (antes: 7 kcal fixo independente do esforço).
- [refactor] Botão 📸 Snap removido — não tinha utilidade prática (o auto-save e o histórico funcionam sem ele).

### Infraestrutura
- Service Worker: `blocos-v2` → `blocos-v3` (limpeza de cache antigo no celular).

### Notas
- localStorage: nenhuma mudança de schema — dados existentes totalmente compatíveis.
- Roadmap próximo: gráficos de tendência de volume de treino ao longo do tempo (volume total por template/sessão).

---

## [v1.1.1] — 2026-02-27

### Melhorado
- [improve] Modal de histórico: valores exibidos em gramas reais (blocos × g/bloco) em vez de número de blocos.
- [improve] Modal de histórico: percentual real sem cap de 100% — ao ultrapassar a meta, exibe o valor verdadeiro (ex: 127%) com cor de aviso laranja.
- [improve] Presets — modo sugestão não-destrutivo: clicar em um preset exibe chips `→N` clicáveis em cada campo P/C/G de cada refeição, sem sobrescrever dados existentes. Barra contextual com "Aplicar tudo" e "✕ Descartar".
- [improve] Presets — Segunda e Sexta separados (antes eram um único "Seg/Sex" com os mesmos valores).
- [improve] Presets — layout 3×2 no mobile (grid, sem quebra assimétrica); accordion fechado por padrão para não ocupar espaço na abertura do app.
- [feat]    Presets editáveis: modal ✏️ full-height com tabs por dia da semana. Usuário edita P/C/G de cada refeição por preset, salva em localStorage e pode restaurar os valores padrão individualmente.
- [improve] Modal de edição de presets: legenda de referência de blocos (1P=Xg / 1C=Xg / 1G=Xg) exibida no topo, refletindo as configurações atuais de ⚙️ Ajustes.

### Notas
- Novo storage key: `blocos_tracker_presets_v1` — armazena presets customizados do usuário. Compatível com dados existentes (não toca nos outros keys).

---

## [v1.1.0] — 2026-02-26

### Adicionado
- [feat] Modal 📋 Histórico de dias na aba Tracker — botão no card de Progresso abre lista full-height com todos os dias registrados, exibindo data, kcal estimada e mini barras P/C/G com % da meta. Badge 📸 marca dias com snapshot manual.
- [feat] Navegação ‹ › entre dias diretamente no date picker do header — limite de 90 dias atrás e +1 dia à frente.
- [feat] Banner contextual "📅 Editando: [data]" com botão "→ Hoje" — aparece no card de Progresso ao navegar para um dia diferente de hoje.
- [docs] Tabela de comandos disponíveis exibida ao final do `/start`.
- [docs] Skill `/end` criada para encerramento estruturado de sessões.

### Corrigido
- [fix] Modal de histórico não carregava dados: `day.meals` é um objeto `{ id: {p,c,g} }`, não array — corrigido uso de `Object.values()`.

### Infraestrutura
- Service Worker atualizado: `blocos-v1` → `blocos-v2` (limpeza de cache antigo).

### Notas
- localStorage: nenhuma mudança de schema — dados existentes totalmente compatíveis.
- Próximo passo natural: iniciar itens do roadmap planejado (dashboard calórico, gráficos de tendência, refeições favoritas, export/import JSON).

---

## [v1.0.0] — 2026-02-26

### Adicionado
- Release inicial: PWA single-file com tracking de macros (Blocos P/C/G), calculadora JP7, medições corporais, treinos com templates e progressão, base de 104 alimentos brasileiros.
- Funciona 100% offline via Service Worker.
- Dados persistidos em localStorage.
