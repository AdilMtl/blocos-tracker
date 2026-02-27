# /undo — Desfazer última alteração

Reverte a última modificação de forma segura.

## Processo

### 1. Avaliar o que desfazer

Verifique o estado:
```bash
git status
git log --oneline -3
```

Apresente as opções:
```
⏪ UNDO — O que desfazer?

OPÇÃO A: Descartar mudanças não commitadas
→ git checkout -- index.html
→ Volta para o último commit

OPÇÃO B: Reverter último commit (já pushado)
→ git revert HEAD
→ Cria um novo commit que desfaz o anterior

OPÇÃO C: Reverter commit específico
→ Qual commit? [mostrar lista dos últimos 5]
```

### 2. Aguardar escolha do usuário

### 3. Executar com cuidado
- Sempre confirme ANTES de executar
- Se reverteu, faça push para atualizar o GitHub Pages
- Confirme o estado final

## Regra de ouro
NUNCA use `git reset --hard` ou `git push --force` sem aprovação explícita. Esses comandos destroem histórico.

$ARGUMENTS
