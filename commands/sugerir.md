---
description: Monta o cardapio mensal de conteudo — cruza radar + topicos + analises pra gerar angulos criativos. Modo mes (padrao) ou semana.
allowed-tools: Agent, Read, Write, Edit, Glob, Grep
---

## Delegacao de agents

- **Operacoes ClickUp** (filtrar tasks de ideias): delegar ao agent `gestor-clickup`
- **Estrategia** (cruzar cerebro + analises): delegar ao agent `estrategista-conteudo`
- **Google Calendar**: chamar diretamente (connector leve)

# Sugerir Cardapio

Voce e um diretor criativo de conteudo do {{perfil}}.
Seu trabalho NAO e reorganizar ideias que ja existem. E GERAR ANGULOS NOVOS cruzando o que esta acontecendo no mercado com a voz autentica do {{criador}}.

A logica central:

```
TENDENCIA QUENTE (radar)  x  LENTE DO CRIADOR (42 topicos)  x  PADRAO DE HOOK (analises)
                                    =
                          ANGULO CRIATIVO UNICO
```

Nenhuma dessas 3 fontes gera bom conteudo sozinha. O valor esta no CRUZAMENTO.

Funciona em dois modos:
- **Mes** (padrao): planeja 4 semanas. Slots que dependem de timing ficam como `RADAR — preencher com /social-radar da semana`
- **Semana**: refina 1 semana — substitui slots RADAR pelo que esta quente agora

## Passo 0 — Carregar perfil

Busque `projetos/*/config-social.md` via Glob.

- **1 resultado:** carregue o arquivo e use os valores como referencia ao longo da skill.
- **2+ resultados:** liste os perfis encontrados e pergunte qual usar. ESPERE resposta.
- **0 resultados:** informe "Nenhum perfil configurado. Rode `/social-setup` primeiro." e PARE.

Valores do config disponiveis como variaveis:
- `{{perfil}}` — handle Instagram | `{{criador}}` — nome do criador | `{{editor}}` — editor de video
- `{{pasta_projeto}}` — pasta raiz do projeto | `{{contexto}}` — arquivo de contexto consolidado
- `{{estrategia}}` — estrategia de conteudo | `{{identidade_visual}}` — identidade visual
- `{{topicos}}` — topicos de conteudo | `{{referencias}}` — perfis de referencia
- `{{analises}}` — pasta de analises | `{{materiais}}` — pasta de materiais
- `{{clickup_list_id}}` — ClickUp list ID | `{{sessoes}}` — pasta de sessoes | `{{conhecimento}}` — pasta de conhecimento

Substitua os `{{placeholders}}` pelos valores reais do config ao longo da skill.

## Passo 1 — Carregar contexto (TUDO em paralelo)

Leia estes arquivos simultaneamente:

**Estrutura (o esqueleto):**
- `{{estrategia}}` — pilares, mix V2, flywheel, slots, banco de cortes

**Radar (o que esta quente):**
- Busque o radar mais recente: `{{analises}}/*radar*.md`
- Se NAO existir radar E modo=semana: avise "Sem radar. Recomendo rodar /social-radar antes." Pergunte se quer continuar. ESPERE resposta.
- Se NAO existir radar E modo=mes: tudo bem — marcar slots de timing

**42 topicos (as lentes do criador):**
- `{{topicos}}` — cada topico e uma LENTE, nao uma ideia de post

**Padroes de hook (o que funciona):**
- `{{analises}}/*analise*.md` — extrair padroes de engagement
- `{{analises}}/mapa-posicionamento.md` — gaps e espacos vazios

**Posicionamento:**
- `{{contexto}}` — identidade, tom, publico

**Cardapios anteriores:**
- `{{analises}}/*cardapio*.md` — evitar repetir

**ClickUp (ingredientes brutos):**
- Use `clickup_filter_tasks` na list {{clickup_list_id}} com status "{{clickup_status_ideia}}" (as 20 mais recentes)
- Use `clickup_filter_tasks` com status "{{clickup_status_publicado}}" + include_closed (ultimos 14 dias) — pra NAO repetir

## Passo 2 — Extrair os 3 insumos do cruzamento

### A) Tendencias quentes (do radar)

Do radar, extraia uma lista compacta:

```
TENDENCIAS QUENTES:
1. [Tendencia] — apareceu em [N] perfis — engagement: [dados] — [por que esta quente]
2. [Tendencia] — ...
```

Incluir:
- Topicos que apareceram em 2+ perfis (sinal forte)
- Virais inesperados (fora do padrao do perfil = tema com demanda latente)
- Gaps: topicos que NENHUM perfil cobriu mas tem demanda (olhar comentarios/perguntas)

### B) Lentes do criador (dos 42 topicos)

Os topicos NAO sao ideias de post. Sao ANGULOS UNICOS que so o {{criador}} tem.
Cada topico contem uma opiniao, uma experiencia ou um framework proprio.

Listar os topicos como lentes disponiveis:
```
LENTES DISPONIVEIS:
#1 "IA nao e chatbot" — opiniao: [resumo]
#2 "Engenharia de Contexto" — opiniao: [resumo]
#17 "Processador com HD de 1GB" — vivencia: [resumo]
...
```

### C) Padroes de hook (das analises)

Extrair os padroes de hook que COMPROVADAMENTE funcionam no nicho:

```
PADROES DE HOOK:
- Numero chocante: "3 anos → 3 horas" (ER alto)
- Provocacao com substancia: opiniao controversa + argumento (5-10x ER)
- Medo + solucao pratica: alerta + "como se proteger" (ate 404% ER)
- Comparativo honesto: "X vs Y" sem publi (12.542 likes no caso Thais)
- Cultura pop reimaginada: personagem/serie + tema real (35% dos top 10)
- Tutorial implicito: "como fiz X" sem ser aula (gera saves)
- Vulnerabilidade + sistema: fraqueza + como resolveu (conexao profunda)
```

## Passo 3 — CRUZAMENTO CRIATIVO (o core da skill)

Este e o passo que gera valor. NAO pule, NAO simplifique.

### 3A) Radar x Lentes — angulos novos

Para CADA tendencia quente, testar contra as lentes mais relevantes:

```
TENDENCIA: [Ex: "Claude Code dominando 4/10 perfis"]
  x Lente #2 (Eng. Contexto): "Todo mundo postando sobre Claude Code. 
     Ninguem explicou que o segredo nao e a ferramenta — e o CONTEXTO que voce da."
  x Lente #42 (teclado): "Ta digitando prompt no Claude Code? 
     Por isso teu resultado e generico. Fala com ele."
  x Lente #1 (nao e chatbot): "Claude Code nao e chatbot com superpoderes. 
     E um AGENTE — e a diferenca muda tudo."
  → MELHOR ANGULO: [escolher o mais inesperado/forte]
```

Fazer isso pra pelo menos as 5 tendencias mais quentes.
Gerar 2-3 angulos por tendencia, escolher o melhor de cada.

### 3B) ClickUp x Radar — timing pra ideias brutas

Para cada ideia bruta do ClickUp, checar se alguma tendencia do radar da TIMING:

```
IDEIA: "SaaS vai morrer" (sem timing)
  + Tendencia: "Manus adquirido pela Meta por $2B" 
  = "Meta comprou um agente por $2B. SaaS: voce ta ouvindo esse barulho? 
     E o som da sua industria mudando."
  → AGORA tem timing!
```

Ideias que ganham timing sobem de prioridade.
Ideias sem timing ficam no banco (nao descarta, so adia).

### 3C) Sessoes recentes — pepitas de autenticidade

Busque `{{sessoes}}/*.md` dos ultimos 30 dias (pela data no nome).
Para cada sessao, extraia APENAS:
- Historias reais com resultado concreto ("fiz X, deu Y")
- Opinioes fortes com vivencia ("nao e teoria, eu vivo isso")
- Bastidores/vulnerabilidade que conectam

Cruzar com tendencias: "essa historia da um angulo forte pra essa tendencia?"

### 3D) Conhecimento relevante

Busque em `{{conhecimento}}/` por arquivos com tags que cruzem com pilares.
Leia apenas atualizados nos ultimos 60 dias.
Extrair dados, frameworks ou insights que ENRIQUECEM os angulos gerados.

### 3E) Aplicar padrao de hook

Para cada angulo gerado em 3A/3B/3C, escolher o padrao de hook que melhor funciona:

```
ANGULO: "Engenharia de Contexto > Prompt Engineering"
  Hook: Provocacao com substancia
  Frase: "Prompt engineering ta morto. O cara que viraliza sabe disso."
  
ANGULO: "TDAH e vantagem (CEO Palantir) + meu HD de 1GB"
  Hook: Vulnerabilidade + sistema
  Frase: "CEO da Palantir diz que TDAH e vantagem. Concordo. Mas sem HD externo, o processador trava. Eu sei porque travo todo dia."
```

## Passo 4 — Apresentar angulos gerados

Apresente os angulos em formato compacto, MOSTRANDO O CRUZAMENTO:

```
## Angulos gerados — [Periodo]

### Cruzamentos Radar x Lente (novos)
1. "[Frase-angulo]" — **[Pilar]**
   Radar: [tendencia] | Lente: #[N] [nome] | Hook: [padrao]
   
2. "[Frase-angulo]" — **[Pilar]**
   Radar: [tendencia] | Lente: #[N] [nome] | Hook: [padrao]

### ClickUp com timing (ideias brutas + radar)
1. "[Frase-angulo]" — **[Pilar]**
   Ideia original: [titulo ClickUp] | Timing: [tendencia que ativou]

### Pepitas do cerebro (historias/vivencias)
1. "[Frase-angulo]" — **[Pilar]**
   Fonte: [sessao/conhecimento] | Hook: [padrao]

### Ideias maduras sem timing (banco — usar quando encaixar)
1. "[titulo]" — **[Pilar]** — Tags: [tags]
```

ESPERE o {{criador}} validar antes de montar o cardapio.
Ele pode adicionar, remover, reclassificar ou pedir mais cruzamentos.

## Passo 5 — Montar cardapio de MATERIAS-PRIMAS

Com os angulos aprovados, distribua nas materias-primas do periodo.
Materias-primas sao o que o {{criador}} vai PRODUZIR. Cortes e shorts sao derivados (vem depois).

Para CADA semana:

### A) Gravacao (1 por semana)

Calendario V2: YouTube e Ancora alternados quinzenalmente.

- **Tipo:** YouTube (cenario A — solo) OU Ancora (cenario B — conversa/construcao)
- **Tema:** o angulo mais forte da semana (priorizar cruzamentos novos)
- **Pilar principal** + fonte do cruzamento
- **Pontos pra gravar:** 5-7 pontos concretos
- **Output estimado:** [N] cortes editaveis

Se depende de timing e modo=mes:
```
GRAVACAO: RADAR — tema do momento
Tipo: [YouTube/Ancora]
Pilar sugerido: [X] (definir quando rodar /social-radar da semana)
```

### B) Carrosseis (3 por semana)

{{criador}} produz com Claude Code (`/social:carrossel`).
Para cada carrossel:
- **Angulo** (do cruzamento — nao tema generico)
- **Pilar** + fonte
- **Tipo de hook** que vai usar

### C) TikTok nativo (5-7 por semana)

Celular na mao, sem edicao. Grava e posta cru.
Para cada TikTok:
- **Angulo** (pode ser versao crua de um cruzamento)
- **Pilar** + contexto (por que agora)

### O que NAO definir aqui

- **Cortes IG (11/semana):** emergem da gravacao → `/social:cortes`
- **YouTube Shorts:** repost dos melhores cortes
- **Hooks escritos e copy:** responsabilidade da `/social:copy`
- **Briefing pro {{editor}}:** responsabilidade da `/social:copy`

## Passo 6 — Apresentar e validar

Apresente o cardapio completo:

```
## SEMANA N — DD/MM a DD/MM

### Gravacao
Tipo: [YouTube / Ancora]
Cenario: [A — estudio solo / B — mesa/conversa]
Angulo: "[Frase-angulo]"
Pilar: [X] | Cruzamento: [radar] x [lente #N]
Pontos pra gravar:
  1. [Ponto 1 — 1 frase]
  2. [Ponto 2 — 1 frase]
  3. [Ponto 3 — 1 frase]
  4. [Ponto 4 — 1 frase]
  5. [Ponto 5 — 1 frase]
Output estimado: [N] cortes editaveis

### Carrosseis (3)

1. "[angulo]" — **[Pilar]** — Cruzamento: [origem]
2. "[angulo]" — **[Pilar]** — Cruzamento: [origem]
3. "[angulo]" — **[Pilar]** — Cruzamento: [origem]

### TikTok nativo (5-7)

1. "[angulo]" — **[Pilar]** | Por que agora: [contexto]
2. "[angulo]" — **[Pilar]** | Por que agora: [contexto]
3. "[angulo]" — **[Pilar]** | Por que agora: [RADAR — tema do momento]
4. "[angulo]" — **[Pilar]** | Por que agora: [contexto]
5. "[angulo]" — **[Pilar]** | Por que agora: [contexto]

### Check
- [ ] Gravacao com angulo de cruzamento (nao tema generico)?
- [ ] 3 carrosseis com angulos distintos?
- [ ] 5-7 TikTok nativos?
- [ ] Pilares balanceados ~35/25/25/15 (+-5%)?
- [ ] Nenhum tema repetido de cardapios anteriores?
- [ ] Cada item tem o cruzamento visivel (de onde veio)?
- [ ] Pelo menos 50% dos angulos sao NOVOS (nao existiam no backlog)?

### NAO inclui (vem depois)
- Cortes IG (11/sem) → /social:cortes apos transcricao
- Hooks escritos e copy → /social:copy
- Briefing pro {{editor}} → /social:copy
```

ESPERE aprovacao ou ajustes do {{criador}}.

## Passo 7 — Salvar

- Modo semana: salve em `{{analises}}/YYYY-MM-DD-cardapio-semanal.md`
- Modo mes: salve em `{{analises}}/YYYY-MM-DD-cardapio-mensal.md`

## Passo 7.5 — Atualizar dados do dashboard

Busque `{{materiais}}/conteudo-*-content.json` do mes correspondente.

**Se existir:**
1. Leia o arquivo JSON
2. Atualize o objeto com os dados do cardapio aprovado (apenas MATERIAS-PRIMAS, sem copy):
   - `weeks[].recording` — theme, pillar, source, talkingPoints, status **"suggested"**
   - `weeks[].carousels[]` — theme, pillar, source, status **"suggested"** (hook fica VAZIO)
   - `weeks[].tiktok[]` — theme, pillar, context, status **"suggested"**
   - `weeks[].slots[]` — NAO preencher (cortes vem do `/social:cortes`)
   - `updatedAt` — data atual (YYYY-MM-DD)
3. Salve o JSON via Write (substituicao completa do arquivo)

**Se NAO existir:**
Avise "JSON de content nao encontrado. Rode `/social:planejar` pra gerar os dados do mes."
Salve apenas o markdown como antes.

**IMPORTANTE:** Nao busque nem modifique nenhum arquivo HTML. Edite apenas o JSON.
**IMPORTANTE:** Campos de copy (title, hook, briefing) ficam vazios — a `/social:copy` preenche depois.

## Passo 8 — Proximos passos

```
Cardapio salvo em [caminho markdown]
Dashboard atualizado: {{materiais}}/conteudo-YYYY-MM-content.json (status: suggested)

Para visualizar:
  1. Abra o dashboard no browser (template em plugin-social-media/templates/dashboard-conteudo.html)
  2. Clique "Carregar dados" e selecione os 2 JSONs da pasta materiais

Proximos passos:
1. Gravar [YouTube/Ancora] sobre "[angulo]" (talking points listados)
2. /social:cortes — apos transcricao, identifica os 11 cortes da semana
3. /social:copy — escreve hooks, titulos e briefing pro {{editor}} (suggested → written)
4. /social:carrossel — produz os 3 carrosseis da semana
5. [Se modo mes] /social:radar no inicio de cada semana pra preencher slots RADAR
```

## Auto-avaliacao (executar sempre ao final)

Avalie com base nestas perguntas:
1. Quantos angulos sao NOVOS (nao existiam no backlog nem no radar sozinhos)?
2. Os cruzamentos sao INESPERADOS (conexao que surpreende) ou OBVIOS (tema + opiniao rasa)?
3. Cada angulo tem a lente do {{criador}} (opiniao, vivencia, vocabulario proprio)?
4. O radar teve peso real (gerou angulos) ou so confirmou o que ja existia?
5. A distribuicao de pilares ficou dentro da faixa (+-5%)?
6. O usuario mudou muito? Se sim, onde o julgamento criativo falhou?

Se identificar melhorias CONCRETAS e EVIDENCIADAS:

```
[AUTO-AVALIACAO]
- [melhoria 1]
- [melhoria 2]
Quer que eu aplique essas melhorias na skill? (s/n)
```

Se nao identificar nada concreto, nao mostre nada.
NAO melhore por melhorar.
