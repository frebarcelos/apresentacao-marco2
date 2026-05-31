# Roteiro de Apresentação — GitHub PR Analyzer

**Tempo limite:** 10–12 minutos · **Meta:** ~10 min  
**Data:** 01/06/2026 · AL0337 · UNIPAMPA  
**Demos obrigatórias:** slide 5 (Bernardo — código ao vivo) · slide 7 (Pedro — pytest) · slide 12 (Diogo — dashboard)

> **Regra:** não ler o slide. Falar sobre o projeto; o slide é só apoio visual.

---

## Distribuição por membro

| Quem      | Slides             | Tempo   |
|-----------|--------------------|---------|
| Frederico | 1 · 2              | ~1 min  |
| Bernardo  | 3 · 4 · 5          | ~2 min  |
| Pedro     | 6 · 7              | ~1:40   |
| Dean      | 8 · 9              | ~1:30   |
| Frederico | 10 · 11            | ~1:20   |
| Diogo     | 12 · 13 · 14       | ~2 min  |
| Frederico | 15                 | ~0:10   |
| **Total** |                    | **~10 min** |

> Frederico fala em dois blocos (abertura + módulos dele). Todos os outros falam em bloco contínuo.

---

## Slides detalhados

---

### Slide 1 — Capa ⏱ 0:20 (acum. 0:20)
**Quem:** Frederico

- *"Bom dia. Somos o grupo do GitHub PR Analyzer — um analisador de Pull Requests com programação funcional e LLMs. Cinco módulos, cinco devs, uma entrega."*

---

### Slide 2 — O Problema ⏱ 0:40 (acum. 1:00)
**Quem:** Frederico

- *"O problema é escala. Repositórios no GitHub acumulam milhares de PRs — o dataset que usamos tem milhões de entradas."*
- Apontar as 3 perguntas do slide: *"Nenhuma ferramenta respondia as três de forma integrada. Foi o que construímos."*
- Passar para Bernardo.

---

### Slide 3 — A Solução ⏱ 0:40 (acum. 1:40)
**Quem:** Bernardo

- Percorrer o diagrama de cima pra baixo: *"A entrada é um CSV ou JSON. O io/ lê de forma lazy. As transforms/ aplicam funções puras. O pipeline/ orquestra via composição de funções. O cache evita re-classificar o mesmo PR. O LLM classifica. A UI exibe."*
- *"A regra central era: transforms/ e pipeline/ zero I/O, zero loops imperativos."*

---

### Slide 4 — Estratégia: Guardrails Primeiro ⏱ 0:40 (acum. 2:20)
**Quem:** Bernardo

- *"O Frederico criou um CLAUDE.md que qualquer IA lia antes de gerar código. Tinha a arquitetura, os imports proibidos, as regras FP001 a FP006."*
- *"Antes da Sprint 1 já tínhamos 50 tasks definidas. Cada um sabia exatamente o que entregar."*
- *"Os pre-commit hooks julgavam o resultado independente de qual IA foi usada — ruff, mypy, check-paradigm AST, cobertura."*

---

### Slide 5 — Ingestão Lazy ⏱ 1:00 (acum. 3:20)
**Quem:** Bernardo  
**⚡ DEMO — abrir `src/pr_analyzer/io/csv_reader.py` no terminal/IDE**

- *"Meu módulo foi o io/. O desafio era ler milhões de linhas sem explodir a RAM."*
- Mostrar o `read_prs` no código real: *"O `yield from` garante que só um registro existe em memória por vez. O PRRecord é um NamedTuple — imutável desde a leitura."*
- *"Rodei com 50 mil PRs. O consumo de RAM ficou flat, não cresceu."*

---

### Slide 6 — Transformações Funcionais Puras ⏱ 0:50 (acum. 4:10)
**Quem:** Pedro

- *"Meu módulo, o transforms/, tem uma restrição dura: zero for, zero while, zero append — qualquer violação bloqueia o commit."*
- Explicar `filter_by_state`: *"Retorna um predicado, não o resultado. Isso permite compor com outras funções sem quebrar pureza."*
- Explicar `count_by_language`: *"Usa reduce com dict merge, sem mutar nada."*
- *"Adicionei heurísticas que classificam PRs por palavras-chave antes do LLM — elimina 10 a 30% das chamadas."*

---

### Slide 7 — Property-Based Testing ⏱ 0:50 (acum. 5:00)
**Quem:** Pedro  
**⚡ DEMO — rodar `pytest src/tests/transforms/ -v` no terminal**

- *"Além dos testes unitários, usei Hypothesis para property-based testing."*
- Mostrar o teste: *"Esse teste garante que a soma dos counts sempre iguala o total de PRs, com qualquer input. O Hypothesis gera centenas de entradas aleatórias — incluindo unicode bizarro."*
- Ao rodar o pytest: *"Cobertura quase 100%, incluindo edge cases que nunca teria pensado manualmente."*

---

### Slide 8 — 4 Classificações por PR ⏱ 0:40 (acum. 5:40)
**Quem:** Dean

- *"Meu módulo, o llm/, classifica cada PR em 4 dimensões."*
- Percorrer a tabela: tipo, natureza, clareza, e a complexidade de revisão que veio da integração com o Gift.
- *"A complexidade é calculada por heurística pura — sem LLM — usando tamanho do diff e arquivos alterados."*
- *"O sistema detecta automaticamente se vai usar Groq na nuvem ou Ollama local."*

---

### Slide 9 — 5h → 12 minutos ⏱ 0:50 (acum. 6:30)
**Quem:** Dean

- *"O problema real: classificar 2 mil PRs sequencialmente levava 5 horas. Inviável."*
- Percorrer a tabela: *"asyncio.gather paralelizou o I/O, 2 a 4 vezes de throughput. As heurísticas cortaram 30% das chamadas. Cache por repositório cortou um terço dos tokens."*
- *"Com tudo junto: 12 minutos na primeira execução, instantâneo nas seguintes pelo cache SQLite."*

---

### Slide 10 — Cache + Pipeline ⏱ 0:40 (acum. 7:10)
**Quem:** Frederico

- *"Meu módulo, o cache + pipeline. A regra do cache é simples: nunca classificar o mesmo PR duas vezes."*
- *"A chave usa SHA-256 com separador nulo — evita colisões entre argumentos."*
- *"O pipeline compõe funções via reduce. Sem um único loop explícito — qualquer nova etapa é só acrescentar à composição."*

---

### Slide 11 — Os Guardrails ⏱ 0:40 (acum. 7:50)
**Quem:** Frederico

- *"O check_paradigm analisa a AST antes de cada commit. Se encontrar for, while ou append nos módulos puros, bloqueia — independente de qual IA gerou o código."*
- *"Da integração com o Gift veio o sanitize e o validate_pr — PRs corrompidos são rejeitados antes de consumir token."*
- *"A cada push, pytest roda no Docker. Cobertura abaixo de 80% bloqueia o merge."*

---

### Slide 12 — Dashboard Streamlit ⏱ 1:10 (acum. 9:00)
**Quem:** Diogo  
**⚡ DEMO — dashboard pré-carregado em `http://localhost:8501`**

- *"Meu módulo é a UI. O dashboard é onde tudo isso se torna navegável."*
- Mostrar ao vivo: escala de PRs, seletor de backend, tabela com coluna complexity, gráficos.
- *"O Claudinho é um comando que lia os commits reais do git e gerava resumos de sprint — mantinha o time alinhado sem reunião de status."*

---

### Slide 13 — Resultados ⏱ 0:25 (acum. 9:25)
**Quem:** Diogo

- *"89% de cobertura geral, 100% nos módulos puros. 12 minutos onde eram 5 horas. 10 hooks ativos. 6 sprints entregues mais a integração com o Gift."*

---

### Slide 14 — Próximos Passos ⏱ 0:20 (acum. 9:45)
**Quem:** Diogo

- *"Os itens abertos são benchmark automático de modelos e model routing adaptativo. Testes end-to-end com o dataset completo."*

---

### Slide 15 — Obrigado! ⏱ 0:15 (acum. 10:00)
**Quem:** Frederico

- Todos visíveis. *"Obrigado. Ficamos à disposição para as perguntas."*

---

## Preparação para perguntas do professor

> O período de Q&A vem logo após. Todos devem estar prontos para responder sobre **seu próprio módulo**.

| Pergunta provável | Quem responde | Resposta-chave |
|---|---|---|
| *"Por que programação funcional para esse problema?"* | Frederico | Testabilidade: funções puras não têm estado, cada função é testável isoladamente. Composição: pipeline construído por composição, sem acoplamento. |
| *"Como vocês garantem que o LLM classifica corretamente?"* | Dean | Não garantimos acurácia 100% — o LLM é probabilístico. As heurísticas cobrem os casos óbvios. A clareza da descrição é validada por critérios léxicos, não só pelo LLM. |
| *"O que são as regras FP001 a FP006 exatamente?"* | Frederico | FP001: proibido `for`. FP002: proibido `while`. FP003: proibido `x[i]=y`. FP004: proibido `.append()/.update()/.pop()`. FP005: proibido `open()` fora de io/ ou ui/. FP006: sem estado global mutável. |
| *"O que acontece se o Ollama cair durante a classificação?"* | Dean | Retry com backoff exponencial em `client.py`. Se falhar todas as tentativas, o PR vai para fallback com classificação vazia e é logado. |
| *"Por que separador nulo no make_cache_key?"* | Frederico | Para evitar colisões: `("a", "bc")` e `("ab", "c")` geram a mesma string concatenada, mas com `\x00` como separador ficam distintas. |
| *"A cobertura de 89% inclui os testes do LLM com chamadas reais?"* | Dean/Pedro | Não — o llm/ usa mocks nos testes. A cobertura real de chamadas ao LLM foi validada manualmente com datasets menores. |
| *"O Claudinho é parte do escopo do projeto?"* | Diogo | É uma feature da UI — um comando do Claude Code que integra com a GitHub API para gerar relatórios de sprint. Está dentro do módulo ui/. |

---

## Checklist pré-apresentação

- [ ] Dashboard **pré-carregado** com dataset pequeno (500 PRs) rodando em `localhost:8501`
- [ ] `src/pr_analyzer/io/csv_reader.py` aberto no editor para Bernardo mostrar
- [ ] Terminal com `pytest src/tests/transforms/ -v` pronto para Pedro rodar
- [ ] `index.html` aberto no browser em **tela cheia (F11)**
- [ ] Ensaio cronometrado feito ao menos uma vez (alvo: 9:30 para ter folga)
- [ ] Todos sabem em qual slide entram e qual é a deixa de passagem

## Contingência

| Problema | Ação |
|---|---|
| Dashboard não sobe | Diogo mostra o código da UI no editor e descreve as features |
| pytest demora muito | Pedro cancela após 10s e mostra a cobertura já gerada (`coverage report`) |
| Tempo estourando | Slides 7 (Property Testing) e 14 (Próximos Passos) podem ser pulados |
| Pergunta técnica fora do módulo | Redirecionar: *"Isso é o módulo do X, ele pode explicar melhor"* |
