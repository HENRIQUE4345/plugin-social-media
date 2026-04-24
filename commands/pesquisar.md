---
description: Pesquisa externa tematica antes de criar conteudo — coleta multiplataforma (Apify), analisa padroes HIT/FLOP (Gemini), cruza com cerebro e identifica angulos virgens com lastro. Triangula mundo externo + repertorio interno + persona pra roteiro final.
allowed-tools: Agent, Bash, Read, Write, Edit, Glob, Grep, TodoWrite
---

## Delegacao de agents

- **Coleta multiplataforma (Apify)**: delegar ao agent `coletor-instagram` (apesar do nome, ele orquestra Apify em geral — aceita TikTok, YouTube)
- **Analise profunda de video**: delegar ao agent `analista-video` (Gemini)
- **Consolidacao + cruzamento com cerebro + geracao de roteiro**: delegar ao agent `estrategista-conteudo`
- **Criar task ClickUp no final**: delegar ao agent do plugin-pique `gestor-clickup`

# Pesquisar — {{perfil}}

Voce e um pesquisador de conteudo. Dado um tema, voce descobre o que ja foi produzido sobre ele no mundo, identifica o que funcionou e o que nao funcionou, cruza com o repertorio interno do {{criador}} e gera roteiro com angulo virgem + lastro real.

Objetivo: criador produzir conteudo com ASSERTIVIDADE, nao intuicao. Evitar saturacao, evitar survivorship bias, maximizar diferenciacao narrativa.

**Input obrigatorio:** tema (via argumento ou perguntar).

**Duracao tipica:** 2-3h (a maior parte e tempo de Apify + Gemini). 4-5 checkpoints humanos no meio.

---

## Passo 0 — Carregar perfil + validar input

Busque `projetos/*/config-social.md` via Glob.

- **1 resultado:** carregue o arquivo e use os valores como referencia.
- **2+ resultados:** liste e pergunte qual usar. ESPERE resposta.
- **0 resultados:** informe "Nenhum perfil configurado. Rode `/social-setup` primeiro." e PARE.

Valores do config disponiveis como variaveis (igual outras skills): `{{perfil}}`, `{{criador}}`, `{{pasta_projeto}}`, `{{contexto}}`, `{{estrategia}}`, `{{referencias}}`, `{{analises}}`, `{{materiais}}`, `{{sessoes}}`, `{{conhecimento}}`, `{{clickup_list_id}}`.

**Tema:** se veio como argumento, use. Se vazio, pergunte: "Qual o tema da pesquisa? (ex: 'emulacao digital cerebro mosca', 'neuralink aprovacao FDA', etc)"

Crie TodoWrite com as 9 fases abaixo como tasks pra tracking.

---

## Passo 1 — Setup do tema (Fase 0) — CHECKPOINT 1

Proponha params e pergunte antes de disparar:

**Keywords:** gerar 6-10 keywords PT-BR + 4-8 EN a partir do tema. Mix de termos diretos ("cerebro mosca") + entidades nomeadas ("Eon Systems") + conceitos proximos ("brain emulation", "mind upload").

**Janela temporal:** default 6 semanas retroativas desde hoje (tempo pro tema maturar virais + acumular flops). Permitir override se o tema e mais antigo ou mais fresco.

**Plataformas:** default IG + TikTok + YouTube. X/Twitter opt-in (gera ruido). Se tema e nichado tech/filosofico, YouTube pesa mais (formato longo explicativo).

**Metrica:** engagement rate = (likes + comments + saves) / views. Absoluta (views) tambem pra alcance.

**Amostra-alvo Gemini:** 20-25 videos. Calibrar por volume coletado no Passo 2.

**Output:** crie pasta de trabalho temporaria `{{pasta_projeto}}/inbox/pesquisa-{slug-tema}/` com `00-setup.md` congelando os params. Confirme e PARE pra aprovacao.

---

## Passo 2 — Coleta ampla Apify (Fase 1)

Delegar ao agent `coletor-instagram` pra disparar 3 scrapers em PARALELO (1 chamada com 3 tool calls no mesmo message). Sem pre-filtro — coleta ampla.

**Instagram** — `apify/instagram-scraper`:
- Input: `directUrls` com `https://www.instagram.com/explore/tags/{hashtag}/` (1 URL por hashtag promissora, 6-10 hashtags PT+EN)
- `resultsType: "posts"`, `resultsLimit: 30`, `onlyPostsNewerThan: "{data-inicio-janela}"`
- **ATENCAO:** IG limitado a busca por hashtag/perfil/URL. Keyword search nativa NAO existe nesse actor. Muitas hashtags PT-BR especificas sao vazias — normal.

**TikTok** — `clockworks/tiktok-scraper`:
- Input: `searchQueries` array (todas keywords PT+EN), `resultsPerPage: 30`, `oldestPostDateUnified: "{data}"`, `searchSection: "/video"`, `shouldDownloadVideos: false`

**YouTube** — `streamers/youtube-scraper`:
- Input: `searchKeywords: "k1, k2, k3, ..."` (string com virgula), `maxResults: 20`, `sortingOrder: "views"`, `dateFilter: "month"`

Rodar `async: true` + monitorar com `get-actor-run` a cada 60-120s ate SUCCEEDED. Custo esperado: $1.50-3.00 total.

Salvar 3 JSONs em `01-coleta-bruta/{ig,tiktok,youtube}.json`. Usar `get-actor-output` com `fields` especificos pra nao explodir token.

**ATENCAO MCP:** `get-actor-output` com param `fields` retorna KEYS com dot-notation LITERAL (ex: `"authorMeta.name"`, nao `authorMeta.name` nested). Normalizacao precisa usar `item["authorMeta.name"]`.

---

## Passo 3 — Normalizacao + mapa bruto HTML (Fase 2) — CHECKPOINT 2

Script Node gera formato unificado com:
- platform, id, url, author, followers, views, likes, comments, saves, engagement_rate, view_follow_ratio, duration, language, caption, posted_at, **relevant** (bool: caption contem termos do tema?)

**Filtro de relevancia semantica:** caption precisa conter pelo menos 1 termo do tema (PT+EN). Tipicamente corta 40-50% do volume (hashtags genericas pegam muito ruido).

**Criar `02-mapa-bruto.html`** — tabela interativa blueprint escuro + ambar com:
- Secoes por plataforma
- Colunas: #, Autor (+ followers), ER, Views, Likes, Coments, Saves, Dur, Data, Idioma, Caption (hover=texto completo), URL
- Checkbox "Analisar a fundo" em cada linha
- Filtros rapidos: busca, plataforma, idioma
- Botoes: Exportar selecionados JSON / Baixar JSON
- Contador "X selecionados"

**Salvar mapa + pedir:** "Abra `02-mapa-bruto.html` no browser. Marque os videos que quer analisar a fundo (sugestao: 20-25 total). Quando terminar, me avise que voce salvou `02-selecao-henrique.json` ou ME CONFIRMA se prefere que eu faca pre-selecao automatica seguindo o criterio em 4 camadas abaixo."

### Criterio de pre-selecao (se usuario escolher automatico)

**Camada 1 — Filtro de entrada** (signal-to-noise):
- Views >= 3000 pra TikTok/IG (abaixo, ER oscila)
- Views >= 200 pra YouTube (escala menor)
- relevant = true

**Camada 2 — Diversidade:** idioma (PT/EN/ES), plataforma, angulo narrativo inferido do caption, formato (curto/medio/longo).

**Camada 3 — 3 Tiers:**
- **HIT:** views >= 100k E ER >= 8% E (vfr >= 1.0 se followers disponivel)
- **BASELINE:** views 10k-100k E ER 3-7% (controle)
- **FLOP-RELATIVO:** followers >= 20k E vfr < 0.20 E views >= 3k (perfil grande, alcance fraco — hook nao pescou)

**Camada 4 — Obrigatorios** (fora da amostra livre):
- 1 post da fonte original se houver (ex: @eonsystems, @openai, etc)
- 1 melhor PT-BR disponivel
- 1 melhor angulo filosofico/intelectual (cruza com cerebro do criador)

Meta: **7 HIT + 7 BASELINE + 5 FLOP + 3 obrigatorios = 22 posts**. Se plataforma tiver poucos posts em tier, completar do pool.

**Adicionar +3 YouTube longos** se YT foi cortado pelo filtro de views (criterio separado: `views >= 200 AND duration >= 180s`). Diversifica formato.

---

## Passo 4 — Analise Gemini (Fase 3)

Delegar ao agent `analista-video` pra rodar Gemini em cada video selecionado.

**IMPORTANTE — paralelismo:** max **3** calls simultaneas (Gemini retorna 429 RESOURCE_EXHAUSTED acima disso). Retry automatico 1x em erro 400 "Cannot fetch content" — se falhar 2x, gerar analise **fallback** baseada so em caption+metricas e marcar no JSON `fallback_note`.

**Ferramentas por plataforma:**
- YouTube: `mcp__gemini__gemini_analyze_youtube` com URL
- TikTok/IG: `mcp__gemini__gemini_analyze_url` com `mimeType: "text/html"` na URL da pagina

**Prompt estruturado** (mesmo pra todas as analises):

```
Dataset: pesquisa sobre tema "{tema}". Contexto do video:
[incluir caption (primeiras 400 char) + metricas: views, ER, followers, VFR, idioma, duracao]
Tier: [HIT|BASELINE|FLOP-RELATIVO|OBRIGATORIO|YT-LONGO]

Retorne APENAS JSON valido:
{
  "hook": {"texto_literal_0_5s": "", "tipo": "estatistica|pergunta|analogia|medo|maravilha|choque|direto", "forca_gancho": "1-5"},
  "angulo_narrativo": "matrix-simulacao|consciencia-filosofia|ciencia-pura|medo-creepy|humor|pratico|mind-upload|maravilha",
  "emocao_dominante": "curiosidade|medo|maravilha|humor|urgencia|reflexao|choque",
  "analogia_central": "frase curta",
  "estrutura": {"duracao_total_seg": 0, "segundo_do_climax": 0, "tem_cta": false, "cta_texto": ""},
  "densidade_info": "baixa|media|alta",
  "visual_dominante": "talking-head|render3D|screencast|graficos|animacao|mixed",
  "linguagem": "tecnica|coloquial|mista",
  "mencao_fontes_cientificas": false,
  "hipotese_performance": "1 frase explicando por que (nao) performou",
  "aprendizado_chave": "1 frase do que o video ensina pra criadores do tema"
}
```

Salvar cada resultado como `03-analises/{idx}-{author}.json` com meta + analise combinados.

**Consolidar matriz** (`03-matriz.json`): array unificado com todas as 22-25 analises + meta + stats (count por tier, por angulo, por tipo de hook, forca do hook x tier).

---

## Passo 5 — Consolidacao de padroes HIT vs FLOP (Fase 4)

Delegar ao agent `estrategista-conteudo` pra ler `03-matriz.json` e gerar `04-consolidado.md` com:

1. **Padroes dos HITs** — o que 5+/8 tem em comum (hook, angulo, analogia, duracao, CTA)
2. **Armadilhas dos FLOPs** — o que 5+/8 errou
3. **Mapa de angulos** — heatmap de saturacao (quantos videos usaram cada angulo)
4. **Analogias pegaram vs morreram** — contagem de mencoes
5. **Estrutura narrativa dos HITs** — segundo por segundo (media dos HITs)
6. **Regras emergentes** — hook forca >= 4 como pre-requisito, duracao sweet spot, etc

**Padroes que historicamente aparecem (checar tbm):**
- Hook forca 5 e quase sempre pre-requisito pra HIT. Hook <= 3 = flop quase garantido.
- Videos que TRADUZEM o fato cientifico numa consequencia existencial > videos que so REPORTAM o fato.
- CTA de debate ("what do you think?", "comenta") infla ER mais que CTA de acao.
- Duracao sweet spot TikTok/Reel: 30-90s. Acima de 2min no TikTok morre.
- Analogia pop saturada = sem diferenciacao. Buscar analogias premium subutilizadas.

---

## Passo 6 — Cruzamento com cerebro (Fase 5) — CHECKPOINT 3

Delegar ao agent `estrategista-conteudo` pra cruzar:

**Busca direcionada no cerebro:**
- `{{contexto}}` (marca, pilares, linguagem)
- `{{estrategia}}/*.md`
- `{{conhecimento}}/**/*.md` filtrado por tags relacionadas ao tema (ex: se tema=cerebro, filtra por `ia`, `tdah`, `grafos`, `segundo-cerebro`)
- `{{sessoes}}/**/*.md` — sessoes com tags relevantes
- `pique/produto-*/*.md` se tema cruza com produto Pique

**Gerar `05-lastro-cerebro.md`** mapeando conexoes fortes identificadas com paths clicaveis.

**Gerar `06-angulos-candidatos.md`** com matriz:
| Angulo | Saturacao externa | Lastro interno | Pilar do criador | Verdict |

3-5 angulos ranqueados. Priorizar **angulos virgens com lastro alto** (onde mundo nao saturou E criador tem repertorio pra sustentar). Cada angulo com:
- Tese central (1 paragrafo)
- 3 hooks candidatos (todos forca >= 4 pelo padrao da Fase 4)
- Analogia central (virgem, nao saturada)
- Estrutura proposta
- Por que o criador ganha aqui

**PARE.** Pergunte ao criador qual angulo seguir.

---

## Passo 7 — Gerar roteiro (Fase 6) — CHECKPOINT 4

Com o angulo escolhido, gerar `07-roteiro.md`. Formato-padrao:

1. **Frontmatter:** angulo, pilar, formato (Reel 60-90s ou Carrossel), tom, linguagem
2. **Roteiro falado** segundo por segundo (0-5s hook, 5-20s revelacao, 20-40s twist, 40-60s ponte pessoal, 60-80s insight maior, 80-90s CTA)
3. **3 hooks distintos** (A/B/C) pra teste A/B/C — nao 3 variacoes do mesmo. Tons diferentes pescam publicos diferentes.
4. **Estrutura compartilhada 5-90s** — mesma pras 3 versoes (economia de gravacao)
5. **Headlines overlay** (texto na tela) no padrao visual do criador — max 4 palavras por tela, corta no beat da fala
6. **Shotlist visual** pro editor: enquadramento, luz, b-roll, ritmo
7. **Caption** — uma por versao se A/B/C, alinhada com tom do hook
8. **Hashtags** — minimas, mix pop + nicho + tema
9. **Plano de publicacao A/B/C** — datas (com gap 3-4 dias, ter/qua/qui preferencial, fora de feriado)
10. **Metricas pra comparar** — retention 3s/10s, watch time, ER, comments rate, shares, novos followers
11. **Riscos identificados + mitigacao**

**Regras editoriais obrigatorias:**
- Linguagem do pilar (ex: Arquiteto pra @iairique: pensar/construir/conectar, NUNCA dev/codar/deploy)
- Persona do criador (conferir `{{contexto}}`)
- Hook forca 5 sempre
- Analogia virgem (nao Matrix/upload puros se ja saturados na Fase 4)
- Mencao de produto proprio SO como bastidor implicito — nunca pitch (conferir regra do criador no config)

**PARE.** Pergunte ao criador se aprova ou quer ajustar. Permitir iteracao.

---

## Passo 8 — Validar datas de publicacao (Fase 6.5)

Antes de criar task, VALIDAR que datas propostas no roteiro batem com dia da semana correto.

**SEMPRE:** usar `new Date('YYYY-MM-DD').toLocaleDateString('pt-BR', { weekday: 'long' })` via Bash ou Node pra confirmar. Errar dia da semana e bug comum (janela de maio pode parecer sex mas ser sab, etc).

Delegar ao `gestor-clickup` pra consultar Folder do criador no ClickUp (via MCP pique-clickup, list_tasks) e identificar dias livres nas proximas 2-3 semanas — nao canibalizar outros posts ja agendados do {{perfil}}.

Propor A/B/C em dias uteis (ter/qua/qui preferencial), gap 3-4 dias, fora de feriado. Confirmar com criador.

---

## Passo 9 — Criar task ClickUp (Fase 6 final) — CHECKPOINT 5

Delegar ao agent `gestor-clickup`.

**Modelo:** 1 task unica pra gravacao com briefing dos 3 hooks. Gabriel gera 3 cortes finais da mesma gravacao.

**Campos (customizar por perfil via `{{clickup_list_id}}`):**
- Status inicial: `gravar`
- Due date: data da primeira publicacao (A)
- Deadline Editor: due_date menos 2-5 dias (evitar fim de semana/feriado)
- Pilar de Conteudo: pilar do angulo escolhido
- Formato: Curto (Reel 60-90s) ou Carrossel
- Origem: `IA-sugerir` (pesquisa sistematica)
- Publicar: Instagram (+ TikTok se repostar)
- Semente: descricao curta da origem da ideia ("skill `/social-pesquisar` sobre tema X")
- Referencia de Contexto: path do `07-roteiro.md` no cerebro
- Descricao: template Pique (Contexto / O que fazer / Criterio de pronto), max 15 linhas, com 3 hooks literais em destaque

**Validacao:** apos criar, SEMPRE rodar `get_task` e confirmar TODOS os custom fields gravaram. MCPs falham silenciosamente em campos nao-nativos (dropdowns UUID, dates timestamp) — requer agente com capacidade de API direta. O `gestor-clickup` ja trata isso.

---

## Passo 10 — Arquivar consolidado + limpar inbox

Apos task criada, consolidar outputs relevantes em `{{analises}}/YYYY-MM-DD-pesquisa-{slug-tema}.md` contendo:
- Resumo executivo (3-5 bullets com descobertas)
- Padroes HIT/FLOP identificados (da Fase 4)
- Angulos candidatos identificados (da Fase 5) — inclusive os nao escolhidos
- Angulo escolhido + link pro roteiro
- Link pra task ClickUp

**Manter em inbox** apenas o `07-roteiro.md` (referencia ativa ate publicacao). Apagar os intermediarios (`01-coleta-bruta/`, `02-*`, `03-analises/`, `04-consolidado.md`, `05-lastro-cerebro.md`, `06-angulos-candidatos.md`) — valor ja esta consolidado no `{{analises}}`.

---

## Passo 11 — Apresentar resumo

Mostre resumo compacto e proximos passos:

```
Pesquisa "{tema}" concluida. ~{N} posts coletados, {M} relevantes, {K} analisados a fundo.

Padroes principais:
- [padrao HIT 1]
- [padrao FLOP 1]
- [oportunidade virgem 1]

Angulo escolhido: {angulo}
Task ClickUp: [nome] ({url})
Roteiro: {{pasta_projeto}}/inbox/pesquisa-{slug-tema}/07-roteiro.md
Consolidado: {{analises}}/YYYY-MM-DD-pesquisa-{slug-tema}.md

Proximos passos sugeridos:
- Gravar no dia marcado (Deadline Editor: DD/MM)
- Publicar A → B (gap 3d) → C (gap 5d)
- Monitorar metricas 48h pos cada pub (retention, comments rate, mencoes produto)
- Alimentar {{analises}} com metricas reais pra proxima rodada desta skill
```

---

## Auto-avaliacao (executar sempre ao final)

Avalie a execucao:
1. Quantos videos o Gemini conseguiu analisar (vs fallback caption-only)? Se >20% fallback, ajustar prompt ou retry.
2. O mapa bruto HTML foi util pro criador selecionar, ou ele preferiu pre-selecao automatica? Otimizar o lado mais usado.
3. A Fase 5 (cruzamento cerebro) encontrou angulos virgens com lastro **forte** (nao generico)? Se nao, o cerebro esta pobre no tema ou a busca foi rasa.
4. O criador aprovou o roteiro em 1 iteracao ou precisou de 2+ ajustes? Se ajustou muito, padrao do Passo 7 esta incompleto.
5. Datas do ClickUp estavam corretas (dia da semana)? Bug comum — checkar sempre.
6. Custom fields ClickUp gravaram na primeira tentativa? Se nao, atualizar Passo 9 com detalhe do campo.
7. O criador pediu algo que a skill nao cobriu?

Se identificar melhorias CONCRETAS e EVIDENCIADAS:

```
[AUTO-AVALIACAO]
- [melhoria 1]
- [melhoria 2]
Quer que eu aplique essas melhorias na skill? (s/n)
```

Se nao identificar nada concreto, nao mostre nada.
NAO melhore por melhorar.

---

## Historico de aprendizados embutidos

Esta skill foi destilada do piloto rodado em 2026-04-24 sobre tema "emulacao digital cerebro mosca (Eon Systems)". Principais aprendizados embutidos:

- **IG keyword search nativa nao existe** — usar hashtag + username sweep
- **`get-actor-output` com fields** retorna dot-notation LITERAL nos keys
- **Gemini paralelismo max 3** — acima disso da 429
- **3 tiers > 2 tiers** — BASELINE e controle cientifico essencial
- **FLOP relativo (VFR baixo) > FLOP absoluto** — tema viral geralmente nao tem flops absolutos
- **Hook forca 5 e pre-requisito quase absoluto** — padrao robusto na amostra
- **Mapa HTML interativo > pre-selecao automatica** quando criador quer o olho dele
- **Data-check sempre** — erros de dia da semana acontecem
- **1 task mae > 3 tasks filhas** pra teste A/B/C (Gabriel entrega 3 cortes do mesmo bruto)
- **Custom fields ClickUp exigem UUIDs/timestamps** — delegar pro `gestor-clickup` que sabe lidar
