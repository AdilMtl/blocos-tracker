# /fix — Corrigir bug ou problema

O usuário vai descrever um problema. Sua tarefa é diagnosticar, explicar a causa, e corrigir.

## Processo

### 1. Diagnóstico
- Leia a descrição do problema em $ARGUMENTS
- Localize o código relevante no index.html
- Identifique a causa raiz (não apenas o sintoma)
- Apresente o diagnóstico:

```
🔍 DIAGNÓSTICO

SINTOMA: [O que o usuário vê]
CAUSA: [Por que está acontecendo]
LOCAL: index.html, linha ~XXX
CORREÇÃO: [O que precisa mudar]
RISCO: [Pode quebrar algo? O que?]
```

### 2. Aguarde confirmação

### 3. Corrija
- Faça a menor alteração possível que resolve o problema
- Não refatore código adjacente "de bônus"
- Confirme a correção feita

### 4. Verifique
- Teste mentalmente o cenário que causava o bug
- Verifique se a correção não quebra outros fluxos

## Bugs comuns neste projeto (consultar primeiro)

1. **Exercícios não aparecem ao clicar template** → Loop destrutivo: loadTreinoDay() sobrescrevendo estado após applyTemplate(). Verificar flag treinoLoaded.
2. **Modal cortado pela bottom-nav** → z-index do modal (301) vs nav (100). Verificar se modal-full está aplicado.
3. **Input perde foco ao digitar** → Evento 'input' causando re-render. Deve ser 'change'.
4. **Closure capturando índice errado em loop** → Falta IIFE wrapper. Usar `((i) => () => ...)(idx)`.
5. **Dados não persistem** → Verificar se saveJSON() está sendo chamado após a modificação.

$ARGUMENTS
