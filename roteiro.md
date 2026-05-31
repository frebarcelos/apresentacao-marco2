# Roteiro de Apresentação — GitHub PR Analyzer

**Tempo limite:** 10–12 minutos · **Meta:** 10 min (deixar 2 min de margem para imprevistos)  
**Data:** 01/06/2026 · AL0337 · UNIPAMPA  
**Demo de código obrigatória** — encaixada no slide 5 (Bernardo) e slide 13 (Diogo)

> **Regra:** não ler o slide. Falar sobre o projeto, o slide é só o apoio visual.

---

## Distribuição por membro

| Quem      | Slides        | Tempo  |
|-----------|---------------|--------|
| Frederico | 1 · 2 · 3 · 4 | ~2 min |
| Bernardo  | 5 · 6         | ~2 min |
| Pedro     | 7 · 8         | ~2 min |
| Dean      | 9 · 10        | ~2 min |
| Diogo     | 11 · 12 · 13  | ~2 min |
| Todos     | 14 · 15 · 16  | ~1 min |

---

## Slides detalhados

---

### Slide 1 — Capa ⏱ 0:20 (acum. 0:20)
**Quem:** Frederico

- Apresentar o grupo e o projeto pelo nome.
- *"Desenvolvemos o GitHub PR Analyzer: um analisador de Pull Requests usando programação funcional e LLMs. Cada membro foi responsável por um módulo específico do sistema."*

---

### Slide 2 — O Problema ⏱ 0:40 (acum. 1:00)
**Quem:** Frederico

- Contextualizar o problema antes de mostrar as perguntas.
- *"Repositórios públicos acumulam milhares de PRs. Analisar isso manualmente é inviável — o dataset que usamos, do Kaggle, tem milhões de entradas."*
- Apontar as 3 perguntas: tipo, natureza, clareza.
- *"Nenhuma ferramenta respondia as três de forma integrada."*

---

### Slide 3 — A Solução ⏱ 0:40 (acum. 1:40)
**Quem:** Frederico

- Explicar o fluxo de cima pra baixo.
- *"A entrada é um CSV ou JSON. O dado passa por io/, que lê de forma lazy sem carregar tudo na RAM. As transforms/ aplicam funções puras. O pipeline/ orquestra tudo via composição de funções. Cache evita re-classificar o mesmo PR. O LLM classifica. A UI exibe."*
- Destacar: *"A regra central era: transforms/ e pipeline/ não podem ter I/O nem loops imperativos."*

---

### Slide 4 — Estratégia: Guardrails Primeiro ⏱ 0:40 (acum. 2:20)
**Quem:** Frederico

- *"A pergunta que nos guiou foi: como cada dev usa a IA que quiser sem quebrar o projeto?"*
- Explicar o CLAUDE.md: *"Criamos um arquivo de regras que qualquer IA lê antes de gerar código — arquitetura, imports proibidos, regras FP001 a FP006."*
- *"Criamos 50 tasks antes da Sprint 1. Cada dev sabia exatamente o que entregar."*
- *"Os hooks de pré-commit julgavam o resultado — independente de qual IA foi usada."*

---

### Slide 5 — Ingestão Lazy ⏱ 1:00 (acum. 3:20)
**Quem:** Bernardo  
**⚡ DEMO DE CÓDIGO — abrir o arquivo `io/csv_reader.py` no terminal**

- *"Meu módulo é o io/. A responsabilidade é ler datasets de milhões de linhas sem explodir a RAM."*
- Mostrar o `read_prs` no código real: *"O `yield from` faz com que só um registro exista em memória por vez. O PRRecord é imutável desde a leitura."*
- *"Rodei com o dataset de 50 mil PRs e o consumo de RAM ficou flat."*

---

### Slide 6 — Como Funcionou na Prática ⏱ 0:40 (acum. 4:00)
**Quem:** Bernardo

- *"Na prática, o Frederico passou o enunciado completo pro Claude Code, que gerou o CLAUDE.md com a arquitetura toda já definida."*
- *"A gente recebeu as tasks antes da Sprint 1 — era só implementar, não descobrir o que fazer."*
- *"Cada um escolheu a IA que queria. O que importava era passar nos hooks."*

---

### Slide 7 — Transformações Funcionais Puras ⏱ 0:50 (acum. 4:50)
**Quem:** Pedro

- *"Meu módulo, o transforms/, tem uma restrição dura: zero for, zero while, zero append. Qualquer violação bloqueia o commit."*
- Explicar `filter_by_state`: *"A função retorna um predicado — não o resultado. Isso permite compor com outras funções."*
- Explicar `count_by_language`: *"Usa reduce com dict merge. Sem mutar nada."*
- *"Adicionei heurísticas que classificam PRs óbvios por palavras-chave antes de chamar o LLM — isso elimina 10 a 30% das chamadas."*

---

### Slide 8 — Property-Based Testing ⏱ 0:50 (acum. 5:40)
**Quem:** Pedro

- *"Além dos testes unitários, usei o Hypothesis para property-based testing."*
- Explicar o teste na tela: *"Esse teste garante que a soma dos counts sempre iguala o total de PRs — independente dos dados de entrada. O Hypothesis gera centenas de inputs aleatórios automaticamente, incluindo unicode bizarro."*
- *"Cobertura final em transforms/ ficou em quase 100%, incluindo edge cases que eu nunca teria pensado manualmente."*

---

### Slide 9 — 4 Classificações por PR ⏱ 0:40 (acum. 6:20)
**Quem:** Dean

- *"Meu módulo, o llm/, classifica cada PR em 4 dimensões."*
- Percorrer a tabela: tipo de projeto, natureza, clareza, e a complexidade de revisão que veio da integração com o Gift.
- *"A complexidade é calculada por heurística — sem chamar o LLM — usando tamanho do diff e arquivos alterados."*
- *"O sistema detecta automaticamente se usar Groq na nuvem ou Ollama local."*

---

### Slide 10 — 5h → 12 minutos ⏱ 0:50 (acum. 7:10)
**Quem:** Dean

- *"O problema inicial era sério: classificar 2 mil PRs sequencialmente levava 5 horas."*
- Percorrer a tabela rapidamente: *"asyncio.gather paralelizou o I/O. O batch adaptativo evita retries caros. As heurísticas cortaram 30% das chamadas. O cache de tipo por repositório cortou um terço dos tokens."*
- *"Com tudo isso junto, ficou em 12 minutos na primeira execução e instantâneo nas seguintes pelo cache SQLite."*

---

### Slide 11 — Cache + Pipeline ⏱ 0:40 (acum. 7:50)
**Quem:** Diogo

- *"O cache usa SHA-256 como chave determinística — mesmos argumentos, mesmo hash, resultado já está no SQLite."*
- *"O pipeline é uma composição de funções via reduce. Sem um único loop explícito."*
- *"O SQLite roda em WAL mode, é thread-safe e sobrevive ao Docker rebuild."*

---

### Slide 12 — Os Guardrails ⏱ 0:40 (acum. 8:30)
**Quem:** Diogo

- *"O check_paradigm é um script que analisa a AST do código antes de cada commit. Se encontrar um for ou while nos módulos puros, bloqueia."*
- *"Da integração com o Gift veio o sanitize e o validate_pr — PRs sem título e sem repo são rejeitados antes de consumir um token do LLM."*
- *"A cada push, o pytest roda no Docker. Cobertura abaixo de 80% bloqueia o merge."*

---

### Slide 13 — Dashboard Streamlit ⏱ 1:00 (acum. 9:30)
**Quem:** Diogo  
**⚡ DEMO — abrir o dashboard no browser: `http://localhost:8501`**

- *"O dashboard é o ponto de entrada do sistema para o usuário final."*
- Mostrar ao vivo: upload, seletor de escala, seletor de backend, tabela com a coluna complexity.
- *"Esse Claudinho aqui não foi à toa — ele lia os commits reais do git e gerava resumos de sprint estilo WhatsApp. Mantinha o time alinhado sem reuniões."*

---

### Slide 14 — Resultados ⏱ 0:20 (acum. 9:50)
**Quem:** Frederico

- Passar rapidamente pela tabela: *"89% de cobertura geral, 100% nos módulos puros, 12 minutos de processamento onde eram 5 horas, 10 hooks ativos, 6 sprints entregues."*

---

### Slide 15 — Próximos Passos ⏱ 0:15 (acum. 10:05)
**Quem:** Frederico

- *"Ainda temos o benchmark automático de modelos e o model routing adaptativo pendentes. São os últimos itens abertos."*

---

### Slide 16 — Obrigado ⏱ 0:10 (acum. 10:15)
**Quem:** Todos

- Todos visíveis. Frederico: *"Obrigado. Ficamos à disposição para perguntas."*

---

## Checklist pré-apresentação

- [ ] Dashboard rodando: `make docker-run` (ou `streamlit run src/pr_analyzer/ui/app.py`)
- [ ] `io/csv_reader.py` aberto no terminal/IDE para Bernardo mostrar
- [ ] Arquivo `index.html` aberto no browser em tela cheia (F11)
- [ ] Todos sabem em qual slide entram
- [ ] Ensaio cronometrado feito ao menos uma vez

## Contingência

| Problema | Ação |
|----------|------|
| Dashboard não sobe | Mostrar o código da UI no editor |
| Tempo estourando | Slides 8 e 15 podem ser pulados |
| Pergunta técnica difícil | "Posso detalhar melhor no período de perguntas" |
