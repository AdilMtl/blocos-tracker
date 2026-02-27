# /improve — Melhorar funcionalidade existente

Melhora algo que já existe: visual, performance, UX, ou qualidade do código. Diferente de /feature (que adiciona algo novo) e /fix (que corrige bug).

## Processo

### 1. Analisar
Leia $ARGUMENTS e identifique o tipo de melhoria:

- **Visual**: Ajustar espaçamentos, cores, tamanhos, alinhamento
- **UX**: Melhorar fluxo de interação, reduzir toques, feedback mais claro
- **Performance**: Reduzir re-renders, otimizar loops, lazy loading
- **Código**: Limpar duplicação, melhorar nomes, organizar seções

### 2. Propor
Apresente a proposta:

```
✨ MELHORIA: [Título]

TIPO: [Visual / UX / Performance / Código]
ONDE: [Qual aba, componente ou seção]
ANTES: [Como está hoje]
DEPOIS: [Como fica]
ALTERAÇÕES: [Quantas linhas, quais seções]
```

### 3. Aguardar aprovação

### 4. Implementar
- Alterações incrementais, uma por vez
- Se é visual, descreva como fica no mobile (375px)
- Se é UX, descreva o fluxo de toque passo a passo

## Regras

- Melhoria não pode quebrar funcionalidade existente
- Melhoria visual deve manter o design system (variáveis CSS)
- Não mude comportamento sem avisar — se algo funcionava de um jeito, manter
- Priorize melhorias que afetam o uso DIÁRIO (tracker, treino) sobre uso esporádico (calc, medição)

$ARGUMENTS
