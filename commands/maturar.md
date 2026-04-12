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

### 1.6 — Analises de concorrentes

- Glob `{{analises}}/**/*.md` — identificar padroes/lentes/formulas empiricas aplicaveis
- Priorizar: `mapa-posicionamento.md`, `padroes-*.md`, analises de perfis de referencia

### 1.7 — Radar da semana (timing externo)

- Glob o arquivo mais recente em `{{pasta_projeto}}/radar/` OU `{{analises}}/radar/` — o que existir
- Se nenhum radar encontrado: seguir sem (fonte C vira opcional)
- Read o radar encontrado: trends quentes, noticias, eventos

## Passo 2 — Cruzar cada pepita com as 4 fontes

Pra CADA pepita, fazer busca targeted **dinamica**:

### Fonte A — Cerebro vivo (semente)

Extrair 2-4 keywords da pepita (titulo + descricao). Rodar Grep em:
- `{{sessoes}}/**/*.md` — brainstorms, downloads, reunioes
- `{{conhecimento}}/**/*.md` — aprendizados, frameworks, referencias
- `{{pasta_projeto}}/**/*.md` — contexto do projeto especifico

Pra cada match relevante, Read o trecho. **Capturar arquivo:linha especifico** — isso vira a semente verificavel. Aceitar como semente valida quando:
- Trecho tem 3+ linhas de contexto do `{{criador}}` sobre o tema (nao so menciona a palavra)
- E opiniao/framework/vivencia — nao apenas citacao externa

### Fonte B — Analises (formula empirica)

Buscar nas analises lidas no Passo 1.6 por:
- Formulas de hook que funcionam pra angulos similares (ex: "revelacao contraintuitiva" pra pepita que ataca senso comum)
- Lentes de analise aplicaveis (ex: "bastidor" vs "resultado" vs "antes/depois")

Se nenhuma formula aplicavel for obvia, deixar esse bloco vazio — nao inventar.

### Fonte C — Radar (timing)

Checar se a pepita conecta com alguma trend/noticia/evento quente do radar:
- Match forte → registrar `timing: quente — {tendencia X}`
- Match fraco ou ausente → `timing: atemporal`

### Fonte D — Historico (anti-repeticao)

Comparar angulo candidato com titulos publicados (Passo 1.2) E Visoes existentes (Passo 1.3):
- Angulo ja feito nas ultimas 4 semanas em publicado → `repetido: true` (sinaliza, nao bloqueia — `{{criador}}` decide se quer re-abordar)
- Visao muito similar ja existe → sugerir mesclar com a Visao existente ao inves de criar nova (no relatorio)

## Passo 3 — Classificar Caso A ou Caso B

### Caso A — Contexto rico (promove pra `Visao`)

Requer TODOS:
1. Pelo menos **1 semente verificavel** da Fonte A (arquivo:linha com contexto real)
2. Angulo nao trivialmente repetido (Fonte D)
3. Cabe em UM pilar claro (o pilar ja vem do `/social-triagem` — nao re-classificar)

Gerar **brief denso** com 7 blocos:

```markdown
## Tese cristalizada
[1-2 linhas. A afirmacao central que a Visao defende. Voz de {{criador}}.]

## Semente
- `arquivo:linha` — [1 linha do que tem la]
- (1-3 sementes. A primeira e a principal — vai no custom field Semente.)

## Angulo proposto
[Como atacar o tema. 2-4 linhas. Ponto de entrada, antagonista, virada.]

## Formato sugerido
[UM formato do catalogo — YouTube | Solo | Dividida | Peguei Gravei | Carrossel]
**Por que:** [1 linha justificando — match com formula empirica + tom]

## Hook-seed
[Uma frase inicial testavel. Nao e copy final — e direcao.]

## Formula empirica validada
[Se Fonte B rendeu: nome da formula + analise:linha. Se nao: "a validar em `/social-sugerir`"]

## Timing
Quente — {tendencia X da semana, expira em {prazo}} | Atemporal
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

#### 1. "[titulo pepita]" (ID: xxx)
- **Tese:** [...]
- **Semente principal:** `arquivo:linha` — [trecho 1 linha]
- **Angulo:** [...]
- **Formato sugerido:** [...]
- **Hook-seed:** "[...]"
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
3. **Angulos foram genuinamente novos?** Alguma Visao criada ja tinha Visao-irma cobrindo angulo identico? (Fonte D falhou?)
4. **Formato sugerido foi coerente?** `{{criador}}` mudou o formato sugerido em >30% das Visoes? Se sim, heuristica de "formula empirica -> formato" precisa ajuste.
5. **Brief denso estava acionavel?** Ao olhar uma Visao criada, `/social-sugerir` vai conseguir clonar angulos especificos sem precisar re-pesquisar cerebro?

Se identificar melhorias CONCRETAS e EVIDENCIADAS:

```
[AUTO-AVALIACAO]
- [melhoria 1]
- [melhoria 2]
Quer que eu aplique essas melhorias na skill? (s/n)
```

Se nao identificar nada concreto, nao mostre nada.
NAO melhore por melhorar.
