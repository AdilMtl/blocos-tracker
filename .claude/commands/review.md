# /review — Revisar código antes de deploy

Faz uma revisão completa do estado atual do index.html verificando qualidade, bugs potenciais e aderência às convenções.

## Checklist de revisão

Execute cada verificação e reporte:

### Integridade estrutural
- [ ] HTML: todas as tags abrem e fecham corretamente
- [ ] JS: sem erros de sintaxe (verificar mentalmente parênteses, chaves, colchetes)
- [ ] CSS: todas as propriedades são válidas
- [ ] IDs referenciados no JS existem no HTML
- [ ] Funções chamadas no INIT estão definidas acima

### Regras críticas do projeto
- [ ] Nenhum `<form>` tag
- [ ] Closures com IIFE em todos os loops com event listeners
- [ ] Inputs de treino usam evento `change` (não `input`)
- [ ] Inputs com font-size >= 16px
- [ ] Modais com z-index adequado (> nav 100)
- [ ] Flag treinoLoaded respeitada

### Mobile
- [ ] Todos os botões têm min-height >= 40px (touch target)
- [ ] Textos legíveis (>= 11px)
- [ ] Nenhum overflow horizontal na tela 375px
- [ ] Modais não ficam atrás da bottom-nav

### Dados
- [ ] Schema do localStorage compatível com versões anteriores
- [ ] saveJSON() chamado após toda modificação de estado
- [ ] loadJSON() com fallback adequado

## Formato do resultado

```
📝 REVIEW

ESTADO: ✅ Limpo / ⚠️ Tem pendências / ❌ Tem problemas

ENCONTRADO:
1. [Severidade: 🔴🟡🟢] [Descrição] (linha ~XXX)
2. ...

RECOMENDAÇÕES:
- [Sugestão de melhoria, se houver]

PRONTO PARA DEPLOY: Sim / Não
```

$ARGUMENTS
