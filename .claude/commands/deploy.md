# /deploy — Fazer deploy das alterações

Commita as alterações e faz push para o GitHub. O GitHub Pages atualiza automaticamente em 1-2 minutos.

## Processo

### 1. Verificar estado
```bash
git status
git diff --stat
```

Apresente o que será deployado:
```
🚀 DEPLOY

ARQUIVOS MODIFICADOS:
- index.html (+XX -YY linhas)
- [outros, se houver]

RESUMO DAS MUDANÇAS:
- [Mudança 1]
- [Mudança 2]
```

### 2. Aguardar confirmação

### 3. Executar
```bash
git add .
git commit -m "[tipo]: [descrição curta]"
git push origin main
```

Tipos de commit:
- `feat:` nova funcionalidade
- `fix:` correção de bug
- `improve:` melhoria visual/UX/performance
- `docs:` alteração em documentação
- `refactor:` mudança de código sem alterar comportamento

### 4. Confirmar
```
✅ Deploy concluído!
├── Commit: [hash curto]
├── Mensagem: [mensagem]
└── URL: https://adilmtl.github.io/blocos-tracker/

⏱️ Aguarde 1-2 minutos para o GitHub Pages atualizar.
No celular: feche e reabra o app para carregar a versão nova.
```

### 5. Se mudança é grande: atualizar cache do SW
Se a alteração é significativa (nova feature, mudança de estrutura), lembre de:
1. Incrementar `CACHE_NAME` em sw.js (blocos-v1 → blocos-v2, etc)
2. Incluir sw.js no commit

$ARGUMENTS
