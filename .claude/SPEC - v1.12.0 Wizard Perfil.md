# SPEC: v1.12.0 — Wizard de Configuração Guiada do Perfil

**Status:** Aprovada ✅
**Data:** 2026-03-02

---

## PROBLEMA / MOTIVAÇÃO

A calculadora atual despeja todos os campos de uma vez:
dados básicos + 7 dobras + objetivo + 5 parâmetros de dieta.
O primeiro acesso é intimidador — o usuário não sabe por onde
começar, preenche meio a meio, e o app fica sem BMR configurado
(banner de aviso na Home aparece indefinidamente).

Problemas concretos:
1. Hint do preset aparece embaixo da tela, em 12px — não é visto
2. Sem "comece por aqui" visível quando não configurado
3. Dobras vazias por padrão — usuário preenche só metade
4. Não há orientação sobre "sem dobras, use estimativa"
5. Goal type + parâmetros avançados expostos juntos, sem contexto

---

## O QUE MUDA

- [ ] Novo modal wizard #calcWizard (4 passos + passo 0, fullscreen mobile)
- [ ] Overlay #calcWizardOverlay (z-index 314)
- [ ] Trigger automático: abre quando CALC_KEY === null (primeiro acesso)
- [ ] Trigger manual: botão "🧭 Configurar perfil" no cabeçalho do accordion
- [ ] Ao completar o wizard: preenche o form existente + chama calcAll()
- [ ] Barra de progresso visual (Passo X de 4) no topo do wizard

---

## OS PASSOS DO WIZARD

### PASSO 0 — Revisão (só para usuários com perfil salvo)

```
"Aqui está o seu perfil salvo:"

♂ Homem · 37 anos · 76 kg · 177 cm
🔴 Cut — Déficit 22%, P 2,2g/kg
Moderadamente ativo

Tem algo que queira atualizar?

[Revisar tudo →]   [Recalcular assim ✅]
```

- "Revisar tudo" → vai para Passo 1 com dados pré-preenchidos
- "Recalcular assim ✅" → chama wizardFinish() direto, fecha wizard

### PASSO 1 — "Quem é você?"
- Botões grandes: [♂ Homem] [♀ Mulher]
- Inputs: Idade | Peso (kg) | Altura (cm)
- Validação leve: peso e altura obrigatórios para avançar

### PASSO 2 — "Você tem medidas de dobras cutâneas?"
- Dois cartões de escolha:
  - [📐 Sim, tenho os 7 pontos]
  - [📊 Não, usar estimativa (Mifflin)]
- Se "Sim": expande 7 inputs de dobras inline
- Se "Não": pula dobras, usa Mifflin-St Jeor no cálculo
- Texto: "Com as dobras, o BMR é mais preciso. Sem elas, usamos
  uma fórmula padrão — funciona bem para a maioria das pessoas."

### PASSO 3 — "Qual é o seu objetivo?"
- 4 cards tocáveis (radio visual), full-width:
  - 🟡 Manutenção — sem déficit, proteína moderada
  - 🔴 Cut — déficit 22%, proteína 2,2g/kg
  - 🟢 Recomp — déficit leve 10%, proteína 2,2g/kg
  - 🔵 Bulk — superávit +10%, carbo elevado
- Card selecionado: borda colorida + ícone de check
- Cada card tem 1 linha de descrição em 12px

### PASSO 4 — "Nível de atividade + Revisão"
- Select de atividade (mesmo #calcActivity)
- Card de resumo: mostra BMR estimado em tempo real
- Botão primário: "Calcular e Salvar ✅"
  → preenche form, chama calcAll(), fecha wizard,
     abre resultado no accordion

---

## COMO O USUÁRIO INTERAGE (mobile, 375px)

**Primeiro acesso:**
1. Abre o app → wizard abre automaticamente no Passo 1
2. Toca [♂ Homem], digita idade/peso/altura → "Próximo →"
3. Toca "📊 Não, usar estimativa" → "Próximo →"
4. Toca card "🔴 Cut" (borda vermelha acende) → "Próximo →"
5. Seleciona "Moderadamente ativo", vê "BMR ≈ 1.820 kcal" no resumo
6. Toca "Calcular e Salvar ✅" → banner da Home some

**Usuário que já configurou:**
1. Abre "🧮 Calculadora de perfil" → clica "🧭 Configurar perfil"
2. Passo 0 abre com resumo do perfil atual
3. "Tem algo que queira atualizar?" → [Revisar tudo] ou [Recalcular assim ✅]
4. Se revisar: passa pelos passos com dados pré-preenchidos

---

## IMPACTO NO CÓDIGO

**CSS:**
- .calc-wizard-overlay (fullscreen, z-index 314)
- .calc-wizard (panel 96dvh, z-index 315, flex column)
- .calc-wizard-step (display:none / display:flex — ativo)
- .calc-wizard-progress (barra de steps/dots)
- .wz-goal-card (card tocável, borda colorida no estado selecionado)
- .wz-choice-card (cartão "Sim/Não" dobras)

**HTML:**
- #calcWizardOverlay + #calcWizard (antes do </body>)
- Botão "🧭 Configurar perfil" substituindo o texto do acc-trigger
- 5 divs .calc-wizard-step[data-step="0".."4"]
- #wizardSummary — div onde JS renderiza dados atuais (passo 0)

**JS:**
- openCalcWizard() — detecta CALC_KEY, define step inicial (0 ou 1)
- closeCalcWizard() — fecha sem salvar
- wizardGoTo(step) — navega entre steps
- renderWizardSummary() — formata dados salvos em texto legível
- wizardFinish() — transfere dados → fillCalcForm() → calcAll()
- Estado interno: objeto wizardData = { sex, age, w, h,
  hasSkinfolds, skinfolds{}, goalType, activity }
- Trigger no INIT: if (CALC_KEY === null) openCalcWizard()

**localStorage:**
- SEM mudança de schema — wizard alimenta o mesmo CALC_KEY via calcAll()

---

## RISCOS

- Usuário fecha wizard no meio → não salvar nada, estado anterior intacto
- Wizard pré-preenchido (passo 0) deve exibir labels legíveis, não valores brutos
- Teclado virtual iOS sobe e esconde inputs → wizard usa overflow-y: auto interno
- z-index: deve ficar abaixo do custom food modal (320) → usar 314/315 ✅

---

## CRITÉRIOS DE FEITO

- [ ] Primeiro acesso: wizard abre automaticamente no Passo 1
- [ ] Passo 0 só aparece para usuário com perfil salvo
- [ ] "Recalcular assim" fecha wizard e atualiza resultados sem passar pelos passos
- [ ] Fluxo completo de 4 passos funciona em 375px sem overflow
- [ ] Opção "sem dobras" calcula BMR via Mifflin corretamente
- [ ] Goal type selecionado aplica os presets no accordion
- [ ] "Calcular e Salvar" fecha wizard, accordion mostra resultados
- [ ] Banner da Home some após configuração
- [ ] Fechar wizard sem completar não altera dados salvos
- [ ] Dados do Passo 0 são legíveis (ex: "Moderadamente ativo", não "1.55")
- [ ] Todos os inputs do wizard têm font-size >= 16px
- [ ] Nenhum \<form\> tag introduzido
