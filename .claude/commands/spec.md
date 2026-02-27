# /spec — Escrever especificação de mudança

O usuário vai descrever o que quer em linguagem natural. Sua tarefa é transformar isso numa mini-especificação clara e validável ANTES de qualquer código.

## Processo

1. **Leia a descrição do usuário** em $ARGUMENTS
2. **Consulte o PROJECT.md** para entender o estado atual
3. **Produza a spec** no formato abaixo
4. **Aguarde aprovação** antes de prosseguir para implementação

## Formato da spec

```
📋 SPEC: [Título curto]

PROBLEMA / MOTIVAÇÃO
O que não funciona ou o que falta hoje.

O QUE MUDA
- [ ] [Mudança 1 — onde e o que]
- [ ] [Mudança 2 — onde e o que]

COMO O USUÁRIO INTERAGE
Passo a passo de como o usuário vai usar a feature no celular.

IMPACTO NO CÓDIGO
- CSS: [o que muda / nada]
- HTML: [o que muda / nada]
- JS: [o que muda / nada]
- localStorage: [schema muda? / nada]

RISCOS
- [Risco 1 e como mitigar]

CRITÉRIOS DE FEITO
- [ ] [Critério 1]
- [ ] [Critério 2]
```

## Regras

- Specs devem ser PEQUENAS — uma feature por vez
- Se a descrição é vaga, faça perguntas antes de especificar
- Se a mudança afeta localStorage schema, SEMPRE alertar sobre migração de dados existentes
- Pense mobile-first: tela pequena, toque, teclado virtual

$ARGUMENTS
