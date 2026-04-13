---
description: Curadoria positiva — cruza cada Pepita limpa com cerebro vivo + analises + radar + historico e promove pra `Visao` quando ha contexto rico. Mantem em `Pepita` com tag `precisa-contexto` quando falta base. Ponte `/social-triagem` -> `/social-sugerir`.
allowed-tools: Agent, Read, Write, Edit, Glob, Grep
---

## Delegacao de agents

- **TODAS as operacoes ClickUp** (filtrar, atualizar status, enriquecer descricao, comentarios, tags): delegar ao agent `gestor-clickup`

# Maturar Pepitas

Voce e um curador de conteudo que transforma pepitas brutas em **Visoes** maduras do {{perfil}}.

Seu objetivo e **curadoria positiva**: pegar cada Pepita limpa (pos-`/social-triagem`) e decidir se ela tem contexto rico suficiente pra virar uma **Visao** — uma lente/tese reusavel que pode gerar N conteudos ao longo do tempo.

**Logica central:**

```
PEPITA BRUTA  x  CEREBRO VIVO (semente)  x  ANALISES (formula)  x  RADAR (timing)  x  HISTORICO (anti-repeticao)
                                              =
                                      VISAO MADURA (com brief denso) — OU — PEPITA EM ESPERA (precisa-contexto)
```

**O que Visao NAO e:** uma task one-shot que vira conteudo especifico. Visao e **lente reusavel**. A task-filha (conteudo especifico com carrossel/Solo/YT) e criada depois pelo `/social-sugerir` via clonagem. Esta skill nao cria tasks novas, so promove/mantem.

**O que esta skill NAO faz:**

- NAO descarta, mescla, nem re-classifica pilar — isso e funcao do `/social-triagem` (rodar antes)
- NAO clona Visao em tasks-filhas `Planejado` — isso e funcao do `/social-sugerir` (rodar depois)
- NAO toca em pepitas fora do status `Pepita` (Visoes ja existentes nao sao re-avaliadas aqui)

## Passo 0 — Carregar perfil

Busque `projetos/*/config-social.md` via Glob.

- **1 resultado:** carregue o arquivo e use os valores como referencia ao longo da skill.
- **2+ resultados:** liste os perfis encontrados e pergunte qual usar. ESPERE resposta.
- **0 resultados:** informe "Nenhum perfil configurado. Rode `/social-setup` primeiro." e PARE.

Valores do config disponiveis como variaveis:
- `{{perfil}}` — handle Instagram | `{{criador}}` — nome do criador | `{{editor}}` — editor de video
- `{{pasta_projeto}}` — pasta raiz do projeto | `{{contexto}}` — arquivo de contexto consolidado
- `{{estrategia}}` — estrategia de conteudo | `{{identidade_visual}}` — identidade visual
- `{{referencias}}` — perfis de referencia
- `{{analises}}` — pasta de analises | `{{materiais}}` — pasta de materiais
- `{{clickup_list_id}}` — ClickUp list ID (List Ideias) | `{{sessoes}}` — pasta de sessoes | `{{conhecimento}}` — pasta de conhecimento
- `{{clickup_status_pepita}}` — status bruto (config: "Status ideia: bruta" → `Pepita`)
- `{{clickup_status_visao}}` — status enriquecido (config: "Status ideia: enriquecida" → `Visao`)
- `{{clickup_status_finalizado}}` — status terminal (config: "Status ideia: pronta" → `Finalizado`)
- `{{cf_pilar_id}}` — custom field ID Pilar de Conteudo | opcoes: `{{pilar_arquiteto}}`, `{{pilar_visao}}`, `{{pilar_tdah_sistema}}`, `{{pilar_pessoal}}`
- `{{cf_origem_id}}` — custom field ID Origem | opcoes: `{{origem_cerebro}}`, `{{origem_radar}}`, `{{origem_conversa}}`, etc (ver config)
- `{{cf_semente_id}}` — custom field ID Semente (short_text)

Substitua os `{{placeholders}}` pelos valores reais do config ao longo da skill.

## Passo 1 — Carregar contexto em paralelo

Execute TODOS em paralelo (uma unica leva de tool calls):

### 1.1 — Pepitas limpas (input)

Delegue ao `gestor-clickup`:
- `list_tasks` com `list_id: {{clickup_list_id}}` e `statuses: ["{{clickup_status_pepita}}"]`
- Se `count > len(tasks)`, paginar (page 0, 1, 2...) ate ter todas
- Pra cada pepita: `get_task` pra ler descricao completa + custom fields (pilar, origem, semente ja preenchidos pela triagem)

Se zero pepitas: encerre com `"Nenhuma Pepita pra maturar. Rode /social-triagem primeiro se tiver pepitas novas no inbox."` e PARE.

### 1.2 — Historico publicado (anti-repeticao)

Delegue ao `gestor-clickup`:
- `list_tasks` com `list_id: {{clickup_list_id}}`, `statuses: ["{{clickup_status_finalizado}}"]`, `include_closed: true`
- Ler so titulo + pilar — suficiente pra detectar angulo ja feito

Se o agent reportar zero resultados, pedir `get_hierarchy` pra confirmar nomes dos statuses da list antes de concluir que nao ha publicados.

### 1.3 — Visoes ja existentes (nao re-avaliar)

Delegue ao `gestor-clickup`:
- `list_tasks` com `list_id: {{clickup_list_id}}`, `statuses: ["{{clickup_status_visao}}"]`
- Anotar temas — evita criar Visao duplicata com pepita similar

### 1.4 — Estrategia + posicionamento

- Read `{{estrategia}}` — pilares, mix, tese macro
- Read `{{contexto}}` — identidade, tom, narrativa do `{{criador}}`

### 1.5 — Catalogo de formatos

- Read `{{pasta_projeto}}/linha-de-producao-conteudo.md` — secao "10 formatos" (ou equivalente). Usado pra sugerir formato por Visao.

### 1.6 — Guia de conteudo + banco de ganchos (OBRIGATORIO)

- Glob `{{pasta_projeto}}/**/guia-conteudo*.md` — se existir, Read integral.
- Extrair: 4 fontes de conteudo, banco de ideias ja classificadas por esforco, **ganchos que funcionam** (categorias: Contrario, Numero, Provocacao, Curiosidade, Confissao, Segredo, Erro), hooks ja formulados com arco narrativo real.
- Este arquivo e o MAIOR acelerador de hook-seed — nao pular.

### 1.7 — Cortes ja gravados do proprio perfil (OBRIGATORIO)

- Glob `{{analises}}/**cortes*.md` OU `{{analises}}/**ancora*.md` — identificar cortes ja gravados do `{{criador}}` com hook + pilar + nota A/B/C + transcricao.
- Cada corte e **bruto potencial** pra virar conteudo editando, nao gravando do zero.
- Mapear: corte -> tema/hook -> nota -> tempo bruto.

### 1.8 — Status atual dos cortes no ClickUp (OBRIGATORIO se 1.7 rendeu)

Pra cada corte identificado no 1.7, delegar ao `gestor-clickup`:
- Buscar em List Producao, List Cortes (se houver), e List Ideias por tasks com hooks/temas correspondentes aos cortes.
- Classificar cada corte em 3 estados:
  - **Postado/Finalizado** -> nao usar como bruto (ja foi consumido; se Visao quiser re-abordar, registrar como angulo novo)
  - **Em producao** (Editar/Revisar/Agendar) -> aguardar, nao sugerir editar
  - **Livre** (sem task ou so em Pepita/Visao) -> disponivel pra edicao

### 1.9 — Analises de concorrentes

- Glob `{{analises}}/**/*.md` — identificar padroes/lentes/formulas empiricas aplicaveis
- Priorizar: `mapa-posicionamento.md`, `padroes-*.md`, analises de perfis de referencia
- **Exigir NUMERO** ao extrair formula (ex: "cultura pop reimaginada 27.4% ER em @diegoalmeida.ia" — nao apenas "cultura pop funciona")

### 1.10 — Radar da semana (timing externo)

- Glob o arquivo mais recente em `{{pasta_projeto}}/radar/` OU `{{analises}}/radar/` — o que existir
- Se nenhum radar encontrado: seguir sem (fonte C vira opcional)
- Read o radar encontrado: trends quentes, noticias, eventos

### 1.11 — Cardapios mensais (anti-repeticao ampliada)

- Glob `{{analises}}/**cardapio*.md` E `arquivo/**cardapio*.md`
- Listar pepitas/Visoes ja mencionadas em cardapios anteriores — sinal de maturidade (nao de repeticao). Uma pepita que aparece em 2+ cardapios ja validou interesse do `{{criador}}`.

## Passo 2 — Cruzar cada pepita com as 5 fontes

Pra CADA pepita, fazer busca targeted **dinamica**.

**REGRA ABSOLUTA — preservar descricao literal:** antes de sintetizar tese/angulo, COPIAR textualmente os trechos mais densos da descricao da pepita (lista de pontos, ganchos sugeridos, conexoes cruzadas ja registradas). A descricao original ja E trabalho do `{{criador}}` — nao re-interpretar em outras palavras, usar as palavras dele.

### Fonte A — Cerebro vivo (semente)

Extrair 2-4 keywords da pepita (titulo + descricao). Rodar Grep em:
- `{{sessoes}}/**/*.md` — brainstorms, downloads, reunioes
- `{{conhecimento}}/**/*.md` — aprendizados, frameworks, referencias
- `{{pasta_projeto}}/**/*.md` — contexto do projeto especifico
- `arquivo/**/*.md` — topicos, contexto v1, cardapios anteriores (muitas teses cristalizadas moram aqui)
- `pique/sessoes/**/*.md` — brainstorms cruzados (ver especialmente `brainstorm-conteudo-cruzado*`)

Pra cada match relevante, Read o trecho. **Capturar arquivo:linha especifico** + **texto literal de 1 frase do cerebro** — isso vira a semente verificavel (vai pro brief SEM parafrasear). Aceitar como semente valida quando:
- Trecho tem 3+ linhas de contexto do `{{criador}}` sobre o tema (nao so menciona a palavra)
- E opiniao/framework/vivencia — nao apenas citacao externa

### Fonte B — Bruto gravado (cortes ja existentes)

Comparar a tese candidata com os cortes levantados no Passo 1.7:
- **Match direto** (corte cobre exatamente a tese) -> registrar `bruto gravado: corte X (arquivo:linha, nota, status ClickUp)`. Formato sugerido vira "editar bruto", nao "gravar novo".
- **Match adjacente** (corte tem pedaco da tese, nao cobre inteira) -> registrar como material complementar/teaser.
- **Sem match** -> registrar "sem bruto, gravacao nova necessaria".

### Fonte C — Analises (formula empirica COM NUMERO)

Buscar nas analises lidas no Passo 1.9 por formulas que funcionam pra angulos similares. **Nao aceitar formula sem numero**: exigir metrica (ex: "cultura pop reimaginada 27.4% ER em @diegoalmeida.ia", "posts opinativos 5-10x acima da media em 5/5 perfis", "@conrado 353k likes em arco vulnerabilidade→sistema").

Se nenhuma formula com numero for obvia, deixar esse bloco como `"a validar em /social-sugerir"` — nao inventar.

### Fonte D — Radar (timing)

Checar se a pepita conecta com alguma trend/noticia/evento quente do radar:
- Match forte → registrar `timing: quente — {tendencia X, expira em Y}`
- Match fraco ou ausente → `timing: atemporal`

### Fonte E — Historico + cardapios (anti-repeticao / validacao)

Comparar angulo candidato com:
- Titulos publicados (Passo 1.2) → se ja feito ultimas 4 semanas, `repetido: true` (sinaliza, nao bloqueia)
- Visoes existentes (Passo 1.3) → se muito similar, sugerir mesclar
- Cardapios mensais (Passo 1.11) → se pepita ja apareceu em 2+ cardapios, registrar `validada pelo criador: sim` (positivo — reduz ansiedade de "pepita nova nao testada")

## Passo 3 — Classificar Caso A ou Caso B

### Caso A — Contexto rico (promove pra `Visao`)

Requer TODOS:
1. Pelo menos **1 semente verificavel** da Fonte A (arquivo:linha com contexto real)
2. Angulo nao trivialmente repetido (Fonte D)
3. Cabe em UM pilar claro (o pilar ja vem do `/social-triagem` — nao re-classificar)

Gerar **brief denso** com 8 blocos:

```markdown
## Citacao da pepita
> "[trecho literal da descricao original — preservar as palavras do {{criador}}, nao parafrasear. 2-4 linhas dos pontos mais densos]"

## Tese cristalizada
[1-2 linhas. A afirmacao central que a Visao defende. Voz de {{criador}}.]

## Semente
- `arquivo:linha` — "[texto literal de 1 frase do cerebro que sustenta a tese — nao parafrasear]"
- (1-3 sementes. A primeira e a principal — vai no custom field Semente.)

## Angulo proposto
[Como atacar o tema. 2-4 linhas. Ponto de entrada, antagonista, virada.]

## Bruto existente
[Se Fonte B rendeu match direto/adjacente: "corte X (arquivo:linha, nota A/B/C, status ClickUp: livre/em producao/postado)". Se match direto + livre: formato vira "editar bruto". Se sem match: "nao ha bruto, gravacao nova necessaria"]

## Formato sugerido
[UM formato do catalogo — YouTube | Solo | Dividida | Peguei Gravei | Carrossel | EDITAR BRUTO existente]
**Por que:** [1 linha justificando — match com formula empirica + tom + disponibilidade de bruto]

## Hook-seed
[Uma frase inicial testavel. Se ha bruto, usar fala literal do proprio {{criador}} transcrita no corte (arquivo:linha da transcricao). Senao, direcao testavel.]

## Formula empirica validada
[Se Fonte C rendeu COM NUMERO: nome da formula + metrica + analise:linha. Se nao: "a validar em `/social-sugerir`". NAO aceitar formula sem numero.]

## Timing
Quente — {tendencia X, expira em Y} | Atemporal | Validada em N cardapios (se Fonte E acusou)
```

### Caso B — Sem contexto rico (mantem em `Pepita`)

Quando falha em qualquer requisito do Caso A (principalmente Fonte A vazia). Output:

- **Comentario** na task: `precisa download/brainstorm sobre {tema especifico} antes de maturar — cerebro nao tem semente ainda`
- **Tag** na task: `precisa-contexto`
- Status fica em `Pepita`

## Passo 4 — Apresentar relatorio em batch

```
## Maturacao {{perfil}} — DD/MM/YYYY

**Pepitas avaliadas:** N
**Caso A (virar Visao):** X  |  **Caso B (precisa-contexto):** Y

---

### A — Promover pra Visao (X)

#### 1. "[titulo pepita]" (ID: xxx) [FLAG: BRUTO GRAVADO / BRUTO ADJACENTE / — ]
- **Citacao:** *"[trecho literal da descricao original]"*
- **Tese:** [...]
- **Semente principal:** `arquivo:linha` — "[texto literal cerebro]"
- **Angulo:** [...]
- **Bruto:** [corte X, nota, status ClickUp] ou "nao ha bruto"
- **Formato sugerido:** [...]
- **Hook-seed:** "[...]"
- **Formula (com numero):** [...]
- **Timing:** [...]

#### 2. "[titulo pepita]" (ID: xxx)
[...]

---

### B — Manter em Pepita, marcar precisa-contexto (Y)

| Pepita | O que falta | Sugestao |
|---|---|---|
| "..." | cerebro nao tem vivencia sobre {tema} | download de 20min antes de re-maturar |
| "..." | so menciona palavra mas sem opiniao do {{criador}} | brainstorm pra cristalizar tese |

---

### Observacoes cruzadas
- [Visao X ja existe e cobre pepita Y — sugerir mesclar?]
- [3 pepitas no pilar Visao — concentracao saudavel ou forcando?]
```

PARE e ESPERE o `{{criador}}` revisar. Ele pode:
- Rejeitar caso A especifico (mover pra B ou manter limpa sem Visao)
- Ajustar tese/angulo/formato de qualquer Visao candidata
- Aprovar tudo (`"pode aplicar tudo"`)
- Descartar pepita especifica (`"pepita #3 deletar"` — delega pra `gestor-clickup`)

NAO execute NADA sem aprovacao explicita.

## Passo 5 — Executar (so o aprovado)

**REGRA HARD:** todas as mudancas ClickUp delegadas ao `gestor-clickup`. NUNCA chamar `mcp__pique-clickup__*` direto.

### Caso A aprovado — promover pra Visao

Pra cada pepita aprovada:

1. **Montar markdown_description** consolidando:
   - Descricao original da pepita (preservar)
   - Linha divisoria `---`
   - Os 7 blocos do brief denso gerados no Passo 3

2. Delegar ao `gestor-clickup`: `update_task` com:
   - `status: "{{clickup_status_visao}}"` — muda `Pepita` -> `Visao`
   - `markdown_description: <brief consolidado>`
   - `custom_fields`:
     - `{{cf_semente_id}}` = `arquivo:linha` da semente principal
     - `{{cf_origem_id}}` = `{{origem_cerebro}}` (default quando semente vem do cerebro) OU conforme a fonte (`{{origem_radar}}` se vem de tendencia quente, `{{origem_conversa}}` se vem de sessao de reuniao, etc)
     - Pilar: NAO tocar (ja foi setado pela triagem)

### Caso B aprovado — marcar precisa-contexto

Pra cada pepita:

1. Delegar ao `gestor-clickup`: `add_comment` com o texto gerado no Passo 3 Caso B
2. Delegar ao `gestor-clickup`: `add_tag` com tag `precisa-contexto` (criar se nao existir — `gestor-clickup` resolve)

### Confirmar cada acao executada

```
[x] "Titulo" promovida pra Visao (ID: xxx) — semente: arquivo:linha
[x] "Titulo" mantida em Pepita — tag precisa-contexto + comentario
```

### Auditoria pos-update (OBRIGATORIA)

**NUNCA confiar em "sucesso reportado" pelo agent.** Apos executar todos os updates, delegar ao `gestor-clickup`:

1. `get_task` de CADA task promovida pra Visao
2. Validar que `custom_fields` tem valores reais — especificamente `{{cf_semente_id}}` nao-vazio
3. Validar que `status` = `{{clickup_status_visao}}`
4. Validar que `markdown_description` contem o brief dos 8 blocos

Se alguma validacao falhar:
- **Semente vazia apos update** -> LIMITACAO CONHECIDA do `pique-clickup-mcp`: o `update_task` nao expoe `custom_fields` (apenas `create_task_full` expoe). Reportar explicitamente: "semente gravada no brief markdown mas NAO no custom field — bug do MCP". Propor: (a) fix no `pique-clickup-mcp`, (b) preenchimento manual, (c) ignorar por agora. NAO esconder o problema no resumo final.
- **Status errado / brief errado** -> re-tentar update daquela task especifica.

So ir pro Passo 6 depois de auditar.

## Passo 6 — Resumo final

```
Maturacao concluida.

Antes: N Pepitas limpas
Depois: X Visoes novas | Y Pepitas com precisa-contexto | Z inalteradas

Proximo passo:
1. /social-sugerir semana — clona Visoes maduras em tasks-filhas `Planejado`
   com angulo especifico por formato
2. (Paralelo) Downloads mentais sobre os temas em Y Pepitas marcadas
   precisa-contexto antes de rodar /social-maturar de novo
```

## Auto-avaliacao (executar sempre ao final)

Avalie com base nestas perguntas:

1. **Taxa Caso A/B bateu expectativa?** T1.2 espera >=50% virando Visao. Se ficou muito abaixo, o filtro de Caso A esta severo demais — ou o cerebro realmente esta seco e precisa de rodada de downloads antes.
2. **Sementes foram verificaveis?** O `{{criador}}` conseguiu abrir cada `arquivo:linha` citado e confirmar que o trecho sustenta a tese?
3. **Descricao literal da pepita foi preservada?** Brief citou texto literal ou re-interpretou em outras palavras? Re-interpretar e falha grave — a descricao ja E trabalho do criador.
4. **Cortes gravados foram cruzados?** Alguma Visao teve corte com nota A/B direto ou adjacente ignorado? Se sim, Fonte B falhou — recomendacao de formato virou "gravar novo" quando deveria ser "editar bruto".
5. **Formulas tiveram NUMERO?** Todas as formulas empiricas citadas tem metrica (ER%, likes, multiplicador)? Formulas genericas sem numero sao cheiro de brief raso.
6. **Angulos foram genuinamente novos?** Alguma Visao criada ja tinha Visao-irma cobrindo angulo identico? (Fonte E falhou?)
7. **Formato sugerido foi coerente?** `{{criador}}` mudou o formato sugerido em >30% das Visoes? Se sim, heuristica "formula empirica + disponibilidade de bruto -> formato" precisa ajuste.
8. **Auditoria pos-update confirmou gravacao real?** `get_task` pos-update validou que `custom_fields` tem valor, ou o agent reportou sucesso falso por limitacao de tooling nao detectada?
9. **Brief denso estava acionavel?** Ao olhar uma Visao criada, `/social-sugerir` vai conseguir clonar angulos especificos sem precisar re-pesquisar cerebro?

Se identificar melhorias CONCRETAS e EVIDENCIADAS:

```
[AUTO-AVALIACAO]
- [melhoria 1]
- [melhoria 2]
Quer que eu aplique essas melhorias na skill? (s/n)
```

Se nao identificar nada concreto, nao mostre nada.
NAO melhore por melhorar.
