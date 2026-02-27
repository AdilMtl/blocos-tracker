# Changelog

Todas as mudanças notáveis do Blocos Tracker são documentadas aqui.
Formato baseado em [Keep a Changelog](https://keepachangelog.com/pt-BR/1.0.0/).

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
