# /end — Encerrar sessão de trabalho

Finaliza a sessão com documentação, versionamento e deploy organizado.

## Processo obrigatório

### 1. Resumo da sessão

Leia o histórico de alterações feitas nesta sessão e apresente:

```
📋 RESUMO DA SESSÃO

DATA: [YYYY-MM-DD]
DURAÇÃO ESTIMADA: [baseado no volume de mudanças]

ALTERAÇÕES REALIZADAS:
1. [Tipo: feat/fix/improve] [Descrição curta]
2. [Tipo: feat/fix/improve] [Descrição curta]

ARQUIVOS MODIFICADOS:
- index.html (+XX -YY linhas)
- [outros se houver]

LINHAS TOTAIS: [antes] → [depois]
```

### 2. Perguntas de versionamento

Faça estas perguntas ao usuário (use seleção, não texto aberto):

**Pergunta 1 — Tipo de release:**
- 🔴 Major (mudança grande, quebra compatibilidade de dados)
- 🟡 Minor (nova feature, tudo compatível)
- 🟢 Patch (correção/melhoria, sem feature nova)

**Pergunta 2 — Nota da sessão:**
- Algo que quer registrar? Decisão tomada, ideia para o futuro, problema pendente?
- (pode pular)

**Pergunta 3 — Deploy agora?**
- ✅ Sim, commit + push agora
- ⏸️ Não, só documentar (commit local sem push)
- ❌ Não commitar nada ainda

### 3. Atualizar CHANGELOG.md

Crie ou atualize o arquivo `CHANGELOG.md` na raiz do repositório:

```markdown
# Changelog

## [vX.Y.Z] — YYYY-MM-DD

### Adicionado
- [feat] Descrição da feature

### Melhorado
- [improve] Descrição da melhoria

### Corrigido
- [fix] Descrição da correção

### Notas
- [nota do usuário, se houver]
- [decisões técnicas relevantes]

---

## [versão anterior] — data anterior
...
```

### 4. Atualizar PROJECT.md (se necessário)

Se as alterações mudaram a arquitetura (nova seção CSS, novo modal, novo schema localStorage, nova seção JS), atualize o PROJECT.md para manter o contexto preciso. Apresente o que vai mudar:

```
📝 ATUALIZAÇÕES NO PROJECT.MD:
- [Seção X]: atualizado de "..." para "..."
- [Nova entrada]: adicionado "..."
```

Aguarde confirmação.

### 5. Executar deploy (se aprovado)

```bash
git add .
git commit -m "[tipo](vX.Y.Z): [resumo das mudanças]"
```

Se o usuário aprovou push:
```bash
git push origin main
```

### 6. Encerramento

```
✅ SESSÃO ENCERRADA

Versão: vX.Y.Z
Commit: [hash curto]
Push: [sim/não]
Changelog: atualizado
PROJECT.md: [atualizado/sem mudanças]

📌 PENDÊNCIAS PARA PRÓXIMA SESSÃO:
- [item 1, se houver]
- [item 2, se houver]
- [nenhuma]

Próxima sessão: /start para retomar
```

## Regras

- NUNCA pule a etapa de resumo — é o registro histórico do projeto
- SEMPRE pergunte sobre versionamento, não assuma
- Se o CHANGELOG.md não existe, crie com header e a primeira entrada
- Se mudou algo no schema do localStorage, registre no changelog como ⚠️ BREAKING
- Pendências devem ser concretas e acionáveis, não vagas

$ARGUMENTS
