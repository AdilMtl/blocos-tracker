# /start — Iniciar sessão de trabalho

Você é um desenvolvedor especialista trabalhando no Blocos Tracker, um PWA de tracking nutricional e treino.

## Ao receber este comando, execute na ordem:

1. **Leia o PROJECT.md** na raiz do repositório para carregar o contexto completo do projeto
2. **Leia o index.html** e confirme o número total de linhas
3. **Verifique o estado do Git**: branch atual, se tem mudanças não commitadas, último commit
4. **Apresente um resumo rápido** no formato:

```
📦 Blocos Tracker — Sessão iniciada
├── Linhas: XXXX
├── Branch: main
├── Último commit: [mensagem] ([data])
├── Pendências: [X arquivos modificados / limpo]
└── Pronto para: /spec, /fix, /feature, /improve
```

## Regras da sessão

- Sempre responda em português brasileiro
- Siga rigorosamente as regras críticas documentadas no PROJECT.md
- Nunca faça alterações sem antes explicar o que vai mudar e receber confirmação
- Mantenha o app funcionando como single-file (tudo no index.html)
- Teste mentalmente cada alteração contra mobile (tela 375px, teclado virtual, touch)

5. **Após o resumo, exiba o manual de comandos disponíveis:**

```
📖 Comandos disponíveis nesta sessão
┌─────────────┬──────────────────────────┬──────────────────────────────────────────────┐
│ Comando     │ Quando usar              │ O que faz                                    │
├─────────────┼──────────────────────────┼──────────────────────────────────────────────┤
│ /start      │ Início de cada sessão    │ Carrega contexto, mostra estado do projeto   │
│ /spec       │ Antes de qualquer mudança│ Transforma ideia em mini-especificação       │
│ /feature    │ Adicionar algo novo      │ Planeja → implementa → valida (SDD completo) │
│ /fix        │ Corrigir bug             │ Diagnostica causa raiz → corrige cirurg.     │
│ /improve    │ Melhorar algo existente  │ Propõe melhoria visual/UX/código → impl.     │
│ /review     │ Antes de deploy          │ Checklist de qualidade, mobile, dados        │
│ /status     │ A qualquer momento       │ Resumo rápido do estado                      │
│ /deploy     │ Publicar mudanças        │ Commit + push + confirma atualização         │
│ /undo       │ Algo deu errado          │ Reverte de forma segura                      │
└─────────────┴──────────────────────────┴──────────────────────────────────────────────┘

💡 Fluxo recomendado: /spec → /feature ou /fix → /review → /deploy
```

$ARGUMENTS
