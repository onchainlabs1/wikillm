# LLM Wiki — Schema

Este repositório é um case de estudo do padrão descrito em https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f,
adaptado para funcionar **sem Obsidian**: apenas markdown puro com links relativos padrão
(`[texto](caminho/arquivo.md)`), navegável em qualquer editor (VS Code, GitHub, terminal).

**Tema do case:** ecossistema de LLMs/IA — modelos, labs, papers, benchmarks, pessoas, técnicas.
Escolhido por ser rico em entidades interligadas, mudar rápido (testa detecção de claims obsoletas)
e ter contradições reais entre fontes (testa detecção de contradições).

## Camadas

- `raw/` — fontes brutas, imutáveis. Artigos, papers, transcrições, notas coladas. Nunca editar
  depois de adicionado; se a fonte mudar, adicionar uma nova versão datada.
- `wiki/` — camada mantida pelo LLM. Único lugar onde síntese e interpretação acontecem.
  - `wiki/entities/` — páginas de pessoas, empresas/labs, modelos específicos (ex: `openai.md`, `gpt-5.md`).
  - `wiki/concepts/` — páginas de técnicas, arquiteturas, ideias (ex: `attention.md`, `rag.md`).
  - `wiki/comparisons/` — páginas que comparam duas ou mais entidades/conceitos lado a lado.
  - `wiki/index.md` — catálogo de todas as páginas da wiki: link + resumo de uma linha + tags.
  - `wiki/log.md` — log append-only de ingests, queries e lints. Formato: `## YYYY-MM-DD — <tipo> — <resumo>`.
  - `wiki/synthesis.md` — síntese viva do panorama geral, revisada a cada poucos ingests.

## Convenções de página

- Frontmatter YAML em toda página nova:
  ```yaml
  ---
  type: entity | concept | comparison
  tags: [tag1, tag2]
  updated: YYYY-MM-DD
  sources: [raw/arquivo1.md, raw/arquivo2.md]
  ---
  ```
- Links entre páginas: sempre relativos padrão markdown, nunca wikilinks `[[...]]` (não temos Obsidian
  resolvendo isso — precisa funcionar em qualquer visualizador).
- Cada claim relevante deve citar a fonte (`raw/arquivo.md`) inline ou em nota de rodapé.
- Se uma nova fonte contradiz algo já escrito, **não sobrescrever silenciosamente**: anotar a
  contradição explicitamente na página (`> ⚠️ Contradição: fonte X diz A, fonte Y diz B`) e registrar
  no log.

## Workflows

### Ingest
Quando eu colar/apontar uma fonte nova:
1. Salvar o conteúdo bruto em `raw/` (nome descritivo + data se relevante).
2. Ler e extrair: entidades, conceitos, claims, números, datas.
3. Atualizar ou criar as páginas afetadas em `wiki/` (uma fonte pode tocar várias páginas).
4. Atualizar `wiki/index.md` com páginas novas/alteradas.
5. Adicionar entrada em `wiki/log.md`.
6. Se a fonte for grande demais para o panorama geral, propor atualização em `wiki/synthesis.md`.

### Query
Quando eu fizer uma pergunta:
1. Buscar nas páginas relevantes de `wiki/` (não nas fontes brutas diretamente).
2. Responder com citações (`[wiki/entities/openai.md]` ou `[raw/fonte.md]`).
3. Se a resposta gerar uma análise nova e reutilizável, oferecer para salvar como página nova.

### Lint
Quando eu pedir um lint (ou periodicamente):
1. Procurar contradições entre páginas.
2. Procurar claims que podem estar obsoletas (datas antigas em campos que mudam rápido).
3. Procurar páginas órfãs (sem nenhum link de entrada) ou sem correspondência em `index.md`.
4. Procurar gaps óbvios de dados (entidade mencionada mas sem página própria).
5. Reportar achados e registrar o lint em `wiki/log.md`, mesmo se nada for encontrado.

## Fora de escopo (por ser case de estudo)

- Sem Obsidian, sem grafo visual, sem Dataview, sem Marp.
- Sem `qmd` ou busca híbrida por enquanto — a wiki é pequena o suficiente para grep/leitura direta.
  Revisitar se o número de páginas crescer muito (~50+) e a busca ficar lenta.
