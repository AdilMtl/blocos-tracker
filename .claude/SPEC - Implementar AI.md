# SPEC — Implementar AI Agent no Kcal.ix
> Documento de instrução para LLM externa gerar a especificação técnica completa.

---

## O que é este documento

Este arquivo guia uma LLM (GPT-4, Gemini Gem, Claude Projects, etc.) a produzir uma **especificação técnica detalhada** para implementar um agente de IA nutricional e de treino no app Kcal.ix — em dois estágios progressivos.

Você **não** está lendo a spec. Você está lendo as **instruções para gerar a spec**.

---

## 1. Documentos que você deve anexar junto a este arquivo

Carregue os seguintes materiais no contexto da LLM antes de enviar o prompt:

| # | O que anexar | Como obter |
|---|---|---|
| 1 | **`Context.md`** (completo) | Arquivo na raiz do projeto |
| 2 | **`CHANGELOG.md`** (completo) | Arquivo na raiz do projeto |
| 3 | **Bloco `AI_SYSTEM_PROMPT`** do `index.html` | Linhas 3998–4090 do index.html |
| 4 | **Bloco `exportAIJson`** do `index.html` | Linhas 4092–4203 do index.html |
| 5 | **Este arquivo** (`SPEC - Implementar AI.md`) | Você está lendo ele agora |

> **Dica de extração:** Abra o `index.html`, busque pela string `AI_SYSTEM_PROMPT` e copie do `const AI_SYSTEM_PROMPT = ` até o fechamento em `};` (linha ~4090). Faça o mesmo para `const exportAIJson`.

---

## 2. Contexto do projeto (resumo para a LLM)

```
App: Kcal.ix (PWA single-file, index.html ~6000 linhas)
Stack: HTML/CSS/JS puro, sem framework, sem backend
Storage: localStorage (chaves blocos_tracker_*)
URL: https://adilmtl.github.io/blocos-tracker/
Usuário-alvo: pessoa física fazendo recomposição corporal
Sistema de macros: "blocos" (porções fixas de P/C/G configuráveis)
Export atual: JSON de 60 dias (nutrição + treino + medições + foodLog)
Prompt atual: coach consultivo que lê o JSON e responde análises
```

---

## 3. Features desejadas (input do produto)

A spec deve cobrir os seguintes agentes/skills, organizados nos dois estágios:

### Estágio 1 — Consultivo (análise e recomendações)

**1.1 Smart Predictor de Peso (Regressão de TDEE Real)**
A IA cruza o histórico de aderência calórica (`nutrition.days.totalKcal`) com a variação de peso (`measurements.weightKg`) para estimar o TDEE Real do usuário — em vez de usar apenas a fórmula configurada no perfil.
- Exemplo de output: *"Você está comendo ~1.800 kcal e seu peso estancou por 12 dias. Meu cálculo indica TDEE real de 2.100 kcal — não 2.400 conforme configurado. Sugiro aumentar o déficit em 150 kcal."*

**1.2 Ghost Training (Progressão de Carga)**
A IA analisa `workouts.days`, compara o mesmo exercício em sessões anteriores e sugere carga/reps para a sessão de hoje respeitando o princípio de Sobrecarga Progressiva.
- Exemplo: *"No último Treino A você fez Supino Máquina com 55 kg × 10 reps. Hoje tente 57 kg ou 12 reps."*

**1.3 Corretor de Desvios ("Furei a Dieta")**
A IA recebe a descrição do alimento/refeição fora do plano, estima o impacto em blocos/kcal e redistribui as metas das refeições restantes do dia para manter o saldo de déficit planejado.
- Exemplo: *"Sorvete Ben & Jerry's (200g) = +2C +3G ≈ +470 kcal. Jantar ajustado: remova 1C e 1G da ceia para compensar."*

**1.4 Correlação Comida × Performance**
A IA cruza `nutrition.days` com `workouts.days` para identificar padrões — ex: queda de carga em treinos precedidos por dias de baixo carboidrato.
- Exemplo: *"Em 8 de 11 dias com <100g de carboidrato pré-treino, sua carga no Supino caiu ≥10%. Sugiro migrar 1 bloco C do jantar para o pré-treino."*

**1.5 Auditor de Qualidade Nutricional (Score de Limpeza)**
A IA lê `foodLog.entries.nome` e classifica cada alimento por qualidade (proteína magra / complexo / ultraprocessado / gordura boa). Gera um score semanal e alerta sobre tendências negativas.
- Exemplo: *"Score semana atual: 72/100. Ultraprocessados subiram de 8% para 22% das entradas vs. semana passada."*

**1.6 Gerador de Lista de Compras Dinâmica**
A IA lê os alimentos mais frequentes do `foodLog` (últimos 15 dias) e estima as quantidades necessárias para a próxima semana com base no consumo médio.
- Exemplo: *"Baseado no seu histórico: 3,5 kg de frango, 2 kg de arroz, 1,2 kg de aveia. Exportar lista?"*

**1.7 Detector de Platô e Diet Break**
A IA monitora `measurements` e detecta quando o peso/cintura estagna por 14+ dias mesmo com aderência ≥80%. Sugere Diet Break (subir para manutenção por 7 dias) com cálculo de kcal de manutenção.

**1.8 Ajuste de "Realidade vs. Teoria"**
A IA compara a perda de peso prevista (pelo saldo calórico acumulado) com a perda real registrada em `measurements` e recalcula o TDEE efetivo, propondo ajuste de metas.
- Exemplo: *"Previsão: −400g na semana. Real: −750g. Seu metabolismo parece 200 kcal acima do configurado. Deseja ajustar as metas para comer mais e manter o ritmo?"*

**1.9 Impacto do Cardio em Tempo Real**
A IA lê `workouts.days.cardio` e traduz o gasto calórico de cada sessão em termos motivacionais: quanto adiantou a meta de peso, como afeta o saldo da semana.
- Exemplo: *"Esse pedal de 50 min (+350 kcal) adiantou sua meta de peso final em ~18 horas."*

---

### Estágio 2 — Agêntico (ações diretas no app)

> ⚠️ Este estágio requer integração com API externa (GPT-4o Vision, Gemini Vision ou similar). A spec deve definir a abordagem de autenticação.

**2.1 Foto do Prato → Inserção Automática**
O usuário fotografa a refeição. A IA estima os alimentos, as quantidades e os macros, e propõe o lançamento no foodLog com um clique de confirmação.

**2.2 Ajuste Automático de Metas**
Baseado no Estágio 1 (TDEE Real, platô, realidade vs. teoria), a IA pode propor e aplicar automaticamente novos valores de metas calóricas/de macros nas configurações do app — mediante confirmação do usuário.

**2.3 Plano de Treino Adaptativo**
Com base na progressão histórica de cargas e na disponibilidade semanal, a IA gera ou ajusta o template de treino do usuário (modifica `workouts.templates` no localStorage).

**2.4 Relatório Semanal Automatizado**
Todo domingo, a IA consolida a semana (aderência, treinos, evolução corporal) em um relatório curto com nota de performance e sugestões para a semana seguinte.

---

## 4. Prompt principal para a LLM

Cole o texto abaixo como mensagem principal após carregar os documentos:

---

```
Você recebeu o contexto técnico completo do Kcal.ix: um app PWA brasileiro
de tracking nutricional e de treino, single-file (index.html), sem backend,
usando localStorage para persistência.

Com base em:
1. Context.md — arquitetura e estrutura de dados
2. CHANGELOG.md — histórico de versões
3. AI_SYSTEM_PROMPT — o prompt consultivo atual e o schema JSON do export
4. exportAIJson — os campos exatos que são exportados hoje
5. A lista de features desejadas descrita neste documento

Crie uma ESPECIFICAÇÃO TÉCNICA DETALHADA (spec) para implementar um
Agente de IA Nutricional e de Treino no Kcal.ix em dois estágios.

A spec deve conter, para cada feature:

─── Por feature (Estágio 1) ───────────────────────────────────────
• Nome e descrição funcional
• Dados necessários do JSON (campos exatos, ex: measurements.weightKg)
• Algoritmo ou lógica sugerida (ex: regressão linear, média móvel)
• Formato do output para o usuário (texto, card, alerta)
• Limitações conhecidas (ex: mínimo de N dias de dados necessários)
• Adições necessárias ao AI_SYSTEM_PROMPT atual

─── Por feature (Estágio 2) ───────────────────────────────────────
• Nome e descrição funcional
• Dependência de API externa (qual modelo, por quê)
• Abordagem de autenticação recomendada:
  opção A: API key do próprio usuário (sem custo para o app)
  opção B: backend proxy (custo centralizado, UX mais simples)
• Fluxo de interação passo a passo (UX no mobile, 375px)
• Campos do localStorage que seriam modificados
• Riscos e mitigações

─── Seção global ──────────────────────────────────────────────────
• Quais campos precisam ser adicionados ao exportAIJson para suportar
  as novas features (especialmente as que usam dados não exportados hoje)
• Ordem de implementação recomendada (do mais simples ao mais complexo)
• Dependências entre features (ex: 1.8 depende de dados de 1.1)
• Um checklist de implementação por estágio

Responda em português brasileiro.
Seja específico: cite nomes de campos reais do JSON, não generalize.
Priorize praticidade: o app é mantido por um desenvolvedor solo.
```

---

## 5. Considerações técnicas importantes (leia antes de gerar a spec)

### Sobre o export atual
- O JSON exportado já cobre 60 dias de dados
- `foodLog` só existe em dias que usaram o painel de alimentos (não cobre lançamentos manuais de blocos)
- `workouts.days.exercicios[].nome` é o `exercicioId` (slug), não o nome amigável — a spec deve considerar isso para o Ghost Training
- Medições (`measurements`) são opcionais: nem todo dia tem peso registrado

### Sobre o Estágio 2 e APIs externas
- O app não tem backend — toda lógica roda no browser do usuário
- Para features com visão computacional (foto do prato), a spec deve recomendar se a API key fica no cliente (digitada pelo usuário nas configurações) ou se justifica criar um proxy simples
- Custo por requisição deve ser estimado para orientar a decisão

### Sobre a implementação no app
- Toda nova UI deve seguir o design system existente (variáveis CSS `--accent`, `--surface2`, `--line`, etc.)
- Inputs com `font-size < 16px` causam zoom no iOS — não fazer
- Não usar tags `<form>`
- Novas features de configuração devem ser salvas em novas chaves localStorage (nunca modificar as existentes: `blocos_tracker_*`)
- Qualquer mudança grande exige bump do `CACHE_NAME` em `sw.js`

### Sobre a priorização
- Features do Estágio 1 que dependem apenas do JSON já exportado são implementáveis imediatamente (apenas melhorar o prompt)
- Features que precisam de novos campos no export precisam de mudança no `exportAIJson`
- Features do Estágio 2 exigem decisão de produto antes de qualquer código

---

## 6. Output esperado da spec

A LLM deve entregar um documento markdown estruturado com:

```
# Spec: AI Agent Nutricional — Kcal.ix

## Resumo executivo
## Estágio 1 — Features Consultivas
  ### 1.1 ...
  ### 1.2 ...
  [...]
## Estágio 2 — Features Agênticas
  ### 2.1 ...
  [...]
## Adições necessárias ao exportAIJson
## Adições necessárias ao AI_SYSTEM_PROMPT
## Ordem de implementação recomendada
## Checklist por estágio
## Riscos e dependências
```

---

---

## 7. Guia de implementação para vibe-coder

> Esta seção é para **quem não é desenvolvedor** mas usa IA para codar.
> Avalia honestamente o que você consegue fazer sozinho, o que precisa de ajuda
> e o que pode dar errado — sem subestimar nem superestimar.

---

### 7.1 Avaliação honesta da sua situação atual

Você já mantém um app de ~6.000 linhas de código, com PWA, localStorage, gráficos, modais e lógica de nutrição — usando Claude Code como copiloto. Isso coloca você **bem acima da média de vibe-coders**.

| Capacidade | Sua situação atual | Impacto na implementação |
|---|---|---|
| Editar código existente | ✅ Confortável (você faz isso hoje) | Estágio 1 via prompts é viável sozinho |
| Debugar erros no browser | ⚠️ Parcial (depende do Claude) | Erros de API vão exigir atenção |
| Criar backend/servidor | ❌ Não faz parte do fluxo atual | Estágio 2 exige decisão aqui |
| Gerenciar segredos/senhas | ❌ Nunca precisou até agora | Ponto crítico — ver seção 7.3 |
| Hospedar além do GitHub Pages | ❌ Nunca precisou até agora | Só necessário no Estágio 2 agêntico |

**Conclusão prática:**
- Estágio 1 (consultivo, só prompt) → **você faz hoje, zero risco**
- Estágio 1 (features no app, código JS) → **você faz com Claude Code, médio risco**
- Estágio 2 (agêntico, API externa, foto) → **você consegue, mas precisa de mais contexto — ver abaixo**

---

### 7.2 Ferramentas recomendadas por estágio

#### Para o Estágio 1 — apenas melhorar o AI_SYSTEM_PROMPT

Nenhuma ferramenta nova. Edite o bloco `AI_SYSTEM_PROMPT` direto no `index.html`
com Claude Code, exatamente como faz hoje.

**Custo:** R$ 0. **Tempo:** 1–2 sessões de trabalho.

---

#### Para o Estágio 1 — features de análise dentro do app (código JS)

Continue usando **Claude Code** (o que você usa agora).
Ferramentas complementares opcionais:

| Ferramenta | Para quê | Custo |
|---|---|---|
| [Cursor](https://cursor.sh) | IDE alternativo ao VS Code com IA embutida — bom para refatorações grandes | US$ 20/mês |
| [Claude Projects](https://claude.ai) | Manter contexto do projeto entre conversas sem precisar recarregar | Plano Pro |
| [Chrome DevTools](https://developer.chrome.com/docs/devtools) | Debugar erros de JS no browser — F12 → Console | Grátis |

**Custo:** praticamente zero além do que já paga. **Tempo:** 3–6 sessões.

---

#### Para o Estágio 2 — features agênticas com API externa

Aqui a complexidade sobe. Você vai precisar escolher **uma das duas arquiteturas**:

##### Opção A — API key do usuário (sem backend) ⭐ Recomendado para começar

O usuário digita a própria chave de API nas configurações do app.
As chamadas vão direto do browser para a API (OpenAI ou Google).

```
[Browser do usuário] ──── HTTPS ────► [API OpenAI / Gemini]
        ↑
   API key salva
   no localStorage
   (só no celular do usuário)
```

**Vantagens:** zero custo seu, zero backend, implementável sozinho com Claude Code.
**Desvantagens:** cada usuário precisa ter sua própria conta na OpenAI/Google.
**Risco de segurança:** baixo — a chave fica no dispositivo do próprio dono.

##### Opção B — Backend proxy (você paga, usuários não precisam de conta)

Você cria um servidor mínimo que recebe requisições do app e repassa para a API.

```
[Browser do usuário] ──► [Seu servidor (Vercel/Cloudflare)] ──► [API OpenAI]
                                    ↑
                             SUA API key fica aqui
                             (nunca exposta ao usuário)
```

**Vantagens:** UX mais simples para o usuário (não precisa de conta OpenAI).
**Desvantagens:** você paga pelas requisições de todos os usuários. Mais complexo de manter.
**Recomendação:** só vale se o app tiver muitos usuários ativos.

---

### 7.3 API keys — como obter e onde guardar (crítico)

> Esta é a parte que mais trip coders não-dev. Leia com atenção.

#### Como obter uma API key

**OpenAI (GPT-4o Vision):**
1. Acesse [platform.openai.com](https://platform.openai.com)
2. Crie uma conta → vá em "API Keys" → clique "Create new secret key"
3. Copie a chave (começa com `sk-...`) — ela só aparece uma vez
4. Adicione créditos em "Billing" (US$ 10 já dá para bastante uso)

**Google Gemini:**
1. Acesse [aistudio.google.com](https://aistudio.google.com)
2. Clique "Get API key" → "Create API key"
3. Copie a chave (começa com `AIza...`)
4. Gemini tem tier gratuito generoso para começar

#### Onde guardar a chave (Opção A — no app do usuário)

O fluxo correto para o Kcal.ix:
- Criar um campo de input nas configurações (viewMais) onde o usuário cola a chave
- Salvar em `localStorage` com uma nova chave dedicada (ex: `blocos_tracker_openai_key`)
- **Nunca** colocar a chave diretamente no código (`index.html`)
- **Nunca** commitar a chave no GitHub

```
❌ ERRADO: const API_KEY = "sk-abc123..."   // exposto no código público
✅ CERTO:  const API_KEY = localStorage.getItem("blocos_tracker_openai_key")
```

#### Custo estimado por uso (OpenAI GPT-4o)

| Feature | Tipo de chamada | Custo estimado |
|---|---|---|
| Análise do JSON (60 dias) | ~8.000 tokens | ~US$ 0,04 por análise |
| Foto do prato (Vision) | imagem + ~500 tokens | ~US$ 0,01–0,03 por foto |
| Corretor de desvios (texto) | ~300 tokens | < US$ 0,001 por uso |

Para uso pessoal diário: **US$ 2–5/mês** seria o teto realista.

---

### 7.4 Hosting — o que precisa e o que não precisa mudar

#### Estágio 1 — não muda nada

O app continua no **GitHub Pages** (gratuito, já configurado).
As chamadas de IA saem do browser do usuário direto para a API.
Nenhum servidor novo é necessário.

#### Estágio 2 — se escolher Opção B (proxy)

Você vai precisar de um servidor mínimo. As opções mais simples para não-devs:

| Plataforma | Dificuldade | Custo | Por quê escolher |
|---|---|---|---|
| [Vercel](https://vercel.com) | ⭐ Mais fácil | Grátis até certo volume | Deploy com 1 clique via GitHub, zero config de servidor |
| [Cloudflare Workers](https://workers.cloudflare.com) | ⭐⭐ Médio | Grátis (100k req/dia) | Ultra-rápido, ótimo para proxy simples |
| [Railway](https://railway.app) | ⭐⭐ Médio | ~US$ 5/mês | Mais flexível, bom para APIs com lógica maior |
| [Supabase Edge Functions](https://supabase.com) | ⭐⭐⭐ Difícil | Grátis | Vale só se também usar banco de dados do Supabase |

**Recomendação para vibe-coder:** comece com Vercel. O fluxo é:
1. Criar pasta `api/` no repositório com um arquivo `proxy.js`
2. Push para o GitHub
3. Conectar o repositório no Vercel (2 cliques)
4. Vercel faz o deploy automaticamente a cada push — igual ao GitHub Pages

---

### 7.5 Banco de dados — você precisa?

**Resposta curta: não, para a maioria das features.**

O Kcal.ix já usa `localStorage` e isso suporta todo o Estágio 1 e boa parte do Estágio 2.

Você só precisaria de banco de dados externo se quiser:
- Sync entre dispositivos (celular + computador)
- Backup na nuvem automático
- Múltiplos usuários com contas

Para esses casos, o mais simples para não-devs:

| Solução | Dificuldade | Custo | Quando usar |
|---|---|---|---|
| [Supabase](https://supabase.com) | ⭐⭐ Médio | Grátis (tier generoso) | Sync multi-device, contas de usuário |
| [Firebase Realtime DB](https://firebase.google.com) | ⭐⭐ Médio | Grátis (tier generoso) | Similar ao Supabase, mais documentação |
| JSON no GitHub Gist | ⭐ Fácil | Grátis | Backup simples, sem sync em tempo real |

**Recomendação:** não implemente banco de dados agora. Mantenha `localStorage`.
Se o sync entre dispositivos virar demanda real, adicione depois.

---

### 7.6 Passo a passo de implementação recomendado

> Ordem que minimiza risco e maximiza resultado visível rápido.

#### Fase 0 — Agora (sem código novo)
- [ ] Gerar a spec com a LLM usando este documento
- [ ] Melhorar o `AI_SYSTEM_PROMPT` com as instruções das features 1.1 a 1.9
- [ ] Testar no Gem/GPT com seu JSON exportado real
- [ ] **Resultado:** todas as 9 features consultivas funcionando via chat externo

#### Fase 1 — Estágio 1 no app (com Claude Code, ~4-8 sessões)
- [ ] Adicionar campos faltantes ao `exportAIJson` (conforme spec gerada)
- [ ] Criar botão "Abrir Coach IA" que abre o Gem/GPT com o JSON já copiado
- [ ] Implementar features que não precisam de API externa (ex: Ghost Training, Detector de Platô) como lógica JS pura no app
- [ ] **Resultado:** insights aparecem diretamente no app, sem sair para chat externo

#### Fase 2 — Estágio 2 básico (com Claude Code + API key do usuário, ~6-10 sessões)
- [ ] Adicionar campo de API key nas configurações (viewMais)
- [ ] Implementar o "Corretor de Desvios" (texto → API → resposta no app)
- [ ] Testar custo real por uso com dados próprios
- [ ] **Resultado:** primeira feature agêntica funcional

#### Fase 3 — Foto do prato (mais complexo, avaliar depois)
- [ ] Depende da Fase 2 estar estável
- [ ] Requer decisão: Opção A (API key do usuário) ou Opção B (proxy)
- [ ] Se Opção B: configurar Vercel antes de começar
- [ ] **Resultado:** feature mais "wow" do produto

---

### 7.7 O que pode dar errado (e como se proteger)

| Risco | Probabilidade | Como mitigar |
|---|---|---|
| API key exposta no GitHub | Alta se não tomar cuidado | Nunca colocar no código; sempre via localStorage |
| Custo de API explodir | Baixa para uso pessoal | Definir limite de uso (`max_tokens`) em cada chamada |
| Feature quebrando dados do usuário | Média no Estágio 2 | Nunca sobrescrever localStorage sem confirmação do usuário |
| App ficando lento com chamadas de API | Média | Sempre mostrar loading state; nunca bloquear a UI |
| CORS error ao chamar API direto do browser | Baixa (OpenAI/Google permitem) | Se ocorrer, mover para proxy Vercel |
| Contexto do Claude Code não caber (6000 linhas) | Alta em sessões longas | Usar `/start` sempre; trabalhar por seções |

---

*Gerado em: 2026-03-01 — Kcal.ix v1.10.1*
