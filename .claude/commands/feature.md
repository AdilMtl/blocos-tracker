# /feature — Adicionar nova funcionalidade

Implementa uma nova feature no Blocos Tracker. Requer uma spec aprovada ou descrição clara.

## Processo obrigatório

### Fase 1: Entender
1. Se não existe spec aprovada, rode /spec primeiro e aguarde aprovação
2. Releia o PROJECT.md para as regras críticas
3. Identifique as linhas exatas do index.html que serão afetadas

### Fase 2: Planejar
Apresente o plano técnico ANTES de editar:

```
🔧 PLANO: [Título]

ALTERAÇÕES:
1. CSS (~linha XXX): [o que adiciona/modifica]
2. HTML (~linha XXX): [o que adiciona/modifica]
3. JS (~linha XXX): [o que adiciona/modifica]

ORDEM DE EXECUÇÃO:
1. [Primeiro passo]
2. [Segundo passo]
3. [Terceiro passo]

CHECKLIST PRÉ-BUILD:
- [ ] Closures com IIFE em loops com event listeners
- [ ] Inputs com font-size >= 16px
- [ ] Modais com z-index > 100 (bottom-nav)
- [ ] Eventos 'change' (não 'input') em campos de treino
- [ ] Nenhum <form> tag adicionado
```

Aguarde confirmação para prosseguir.

### Fase 3: Implementar
1. Faça as alterações na ordem planejada
2. Cada alteração deve ser cirúrgica — mude apenas o necessário
3. Após cada bloco de alterações, confirme o que foi feito

### Fase 4: Validar
Após implementar, execute a checklist:
- [ ] JS sem erros de sintaxe
- [ ] Número total de linhas coerente
- [ ] Nenhuma referência quebrada (IDs removidos, funções ausentes)
- [ ] Testou mentalmente o fluxo mobile (toque, scroll, teclado)
- [ ] localStorage schema compatível com dados existentes

## Regras

- NUNCA pule a fase de planejamento
- Uma feature por vez — se é grande, divida em etapas
- Preserve o estilo visual existente (cores, fontes, espaçamentos)
- Sempre adicione CSS ANTES do comentário `/* DESKTOP */`
- Sempre adicione HTML na view correspondente
- Sempre adicione JS na seção temática correta (não jogue tudo no INIT)

$ARGUMENTS
