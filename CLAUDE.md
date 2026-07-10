# LLM Wiki — Schema

Case de estudo do padrão descrito em https://gist.github.com/karpathy/442a6bf555914893e9891c11519de94f,
implementado **100% no GitHub, sem Obsidian**. A camada de navegação (que no gist é o Obsidian) é o
**GitHub Wiki** nativo do repositório — renderizado com sidebar + busca no próprio github.com.

**Tema do case:** **LLMs / Gen AI agents no contexto enterprise** — empresas e produtos de
infraestrutura de agentes (memória, conhecimento, retrieval, orquestração), técnicas, pessoas,
benchmarks e padrões de adoção corporativa. Escolhido por ser rico em entidades interligadas,
mudar rápido (testa detecção de claims obsoletas) e ter contradições reais entre fontes
(testa detecção de contradições).

## Arquitetura de dois repositórios

O padrão usa **dois repos git** (o GitHub Wiki é sempre um repo separado):

| Repo | Git remote | Clone local | Conteúdo |
|------|-----------|-------------|----------|
| Principal | `github.com/onchainlabs1/wikillm.git` (branch `main`) | `/Users/fabio/Desktop/llm-wiki` | `CLAUDE.md` (este schema) + `raw/` (fontes brutas) |
| Wiki | `github.com/onchainlabs1/wikillm.wiki.git` (branch `master`) | `/Users/fabio/Desktop/wikillm-wiki` | páginas sintetizadas mantidas pelo LLM |

Um ingest normalmente **toca os dois repos** (salva a fonte no principal, atualiza páginas no wiki) e faz
**dois commits/pushes**. Isso é o overhead assumido dessa escolha (parte do que o case estuda).

## Camadas

- `raw/` (repo principal) — fontes brutas, imutáveis. Artigos, papers, transcrições, notas coladas.
  Nunca editar depois de adicionado; se a fonte mudar, adicionar nova versão datada.
- **GitHub Wiki** (repo wiki) — única camada onde síntese e interpretação acontecem:
  - `Home.md` — landing + **índice/catálogo** de todas as páginas (link + resumo de uma linha).
  - `_Sidebar.md` — navegação lateral, agrupada por tipo. Manter em sincronia com as páginas.
  - `_Footer.md` — rodapé com links pro repo principal e `raw/`.
  - `Log.md` — log append-only de ingests, queries e lints. Formato: `## YYYY-MM-DD — <tipo> — <resumo>`.
  - `Synthesis.md` — síntese viva do panorama geral, revisada a cada poucos ingests.
  - Páginas de conteúdo: uma por entidade/conceito/comparação (ver convenções abaixo).

## Convenções de página (GitHub Wiki)

O GitHub Wiki tem regras próprias, diferentes do markdown de repo comum. **Seguir estas:**

- **Namespace plano, sem subpastas.** Nome do arquivo = nome limpo da página, ex: `OpenAI.md`,
  `GPT-5.md`, `RAG.md`, `OpenAI-vs-Anthropic.md`. O tipo (entity/concept/comparison) é indicado na
  linha de metadados e no agrupamento da sidebar, **não** em subpasta.
- **Sem YAML frontmatter.** O Wiki renderiza `---` como linha horizontal e mostra o YAML como texto.
  Em vez disso, logo abaixo do `# Título`, uma **linha de metadados em itálico**:
  ```
  *type: entity · tags: lab, us · updated: 2026-07-10 · sources: raw/arquivo.md*
  ```
- **Links entre páginas do wiki:** markdown padrão sem `.md`, apontando pelo nome da página:
  `[OpenAI](OpenAI)`, `[RAG](RAG)`. (Evitar wikilinks `[[...]]` para manter as páginas legíveis
  fora do renderizador do GitHub.)
- **Links para fontes brutas:** apontam pro repo principal via URL absoluta, ex:
  `[fonte](https://github.com/onchainlabs1/wikillm/blob/main/raw/arquivo.md)`.
- Cada claim relevante deve citar a fonte inline.
- Se uma nova fonte contradiz algo já escrito, **não sobrescrever silenciosamente**: anotar a
  contradição na página (`> ⚠️ Contradição: fonte X diz A, fonte Y diz B`) e registrar no `Log.md`.

## Workflows

### Ingest
Quando eu colar/apontar uma fonte nova:
1. Salvar o conteúdo bruto em `raw/` no **repo principal** (nome descritivo + data se relevante).
2. Ler e extrair: entidades, conceitos, claims, números, datas.
3. No **repo wiki**: atualizar/criar as páginas afetadas (uma fonte pode tocar várias).
4. Atualizar o catálogo em `Home.md` e a navegação em `_Sidebar.md`.
5. Adicionar entrada em `Log.md`.
6. Commit + push nos dois repos que foram tocados.
7. Se a fonte for grande demais para o panorama, propor atualização em `Synthesis.md`.

### Query
Quando eu fizer uma pergunta:
1. Buscar nas páginas do wiki (não nas fontes brutas diretamente).
2. Responder com citações (nome da página do wiki ou `raw/fonte.md`).
3. Se a resposta gerar análise nova e reutilizável, oferecer para salvar como página nova.

### Lint
Quando eu pedir um lint (ou periodicamente):
1. Procurar contradições entre páginas.
2. Procurar claims possivelmente obsoletas (datas antigas em campos que mudam rápido).
3. Procurar páginas órfãs (sem link de entrada) ou fora do catálogo/sidebar.
4. Procurar gaps óbvios (entidade mencionada mas sem página própria).
5. Reportar achados e registrar o lint em `Log.md`, mesmo se nada for encontrado.

## Fora de escopo (por ser case de estudo)

- Sem Obsidian, sem grafo visual, sem Dataview, sem Marp — a navegação é a sidebar + busca do GitHub Wiki.
- Sem `qmd` ou busca híbrida por enquanto — a wiki é pequena o suficiente para grep/leitura direta.
  Revisitar se passar de ~50 páginas.
