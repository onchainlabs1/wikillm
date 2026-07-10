# Fonte: Letta (ex-MemGPT) — arquitetura de memória de agente

- **Fontes oficiais consultadas (2026-07-10):**
  - https://www.letta.com (homepage)
  - https://docs.letta.com/guides/agents/memory-blocks/ (Memory blocks / core memory)
  - https://docs.letta.com/guides/agents/archival-memory/ (Archival memory)
  - https://docs.letta.com/concepts/memory-management/ (Understanding memory management)
  - https://docs.letta.com/letta-memgpt (MemGPT)
- **Data de captura:** 2026-07-10
- **Proveniência:** extração via WebFetch/WebSearch das páginas oficiais (os docs são renderizados por
  JS e não capturam verbatim via curl). Conteúdo abaixo é extração/paráfrase fiel das fontes citadas,
  não scrape literal. Reverificar nas URLs acima.

> Fonte bruta imutável. Não editar. A síntese fica nas páginas do GitHub Wiki.

---

## Empresa / posicionamento

- Letta é um **AI research lab** em São Francisco, fundado pelos criadores do **MemGPT** (UC Berkeley,
  Sky Computing Lab). Visão: "machines that learn" — agentes experienciais que "lembram de tudo,
  aprendem continuamente e melhoram com o tempo".
- Produto: **Letta Agent** / **Letta Code**, instalável via `npm i -g @letta-ai/letta-code`. Foco em
  agentes cuja "memória, identidade e capacidades evoluem com a experiência".
- **Origem MemGPT:** trabalho original de "virtual context management" (out/2023); Letta é "a pesquisa
  MemGPT transformada em runtime de agente completo".
- **Clientes enterprise (case studies):** Bilt, 11x, Kognitos, Hunt Club.

## Linhas de pesquisa citadas na homepage

- **Context Repositories** (fev/2026): "gerenciamento programático de contexto e **versionamento baseado
  em git**".
- **Memory Models** (jun/2026): aprendizado do agente via memory models treinados com "memory-native RL".
- **Sleep-time Compute** (abr/2025): o agente raciocina sobre o contexto em tempo ocioso, não só na
  inferência.
- **Continual Learning** (dez/2025): "aprender em token space é a chave" para melhorar agentes.

## Arquitetura de memória (docs oficiais)

Agentes Letta são **stateful**: mantêm memória estruturada que persiste entre conversas e é atualizada
dinamicamente. Diferente de apps LLM tradicionais onde o *código* gerencia estado, aqui o **agente
gerencia a própria memória** via tools (ler/escrever/buscar).

### Core memory / Memory blocks
- **Memory blocks**: segmentos de texto **fixados na janela de contexto** (context window), estruturados
  e rotulados (ex: goals, preferences, persona), **sempre visíveis, sem necessidade de retrieval**.
- Implementados prependando ao prompt do agente em formato tipo XML.
- **Editáveis pelo próprio agente** (self-editing memory): o agente atualiza a própria persona e os
  fatos sobre o usuário conforme aprende. Podem ser **compartilhados entre agentes**.

### Recall memory
- Histórico de conversa **pesquisável**, armazenado fora do contexto (analogia: "disk cache").

### Archival memory
- **Tabela em vector DB**: banco **semanticamente pesquisável** onde o agente guarda fatos, conhecimento
  e dados externos para retrieval de longo prazo. Diferente dos memory blocks (sempre visíveis),
  archival é acessado sob demanda via tool call.

### Analogia de camadas (MemGPT)
- **Core memory** ≈ RAM (na janela de contexto, lida/escrita direto pelo agente).
- **Recall memory** ≈ disk cache (histórico pesquisável fora do contexto).
- **Archival memory** ≈ cold storage (armazenamento de longo prazo via tool call).

O framework gerencia o loop do agente entre as três camadas; o agente decide o que trazer para o
contexto a cada momento (virtual context management, herdado do MemGPT).
