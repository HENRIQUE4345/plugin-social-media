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

**YouTube (validacao de demanda):**
- Busque `{{conhecimento}}/*youtube*` ou `{{conhecimento}}/*yt*` via Glob
- Se existir: usar dados de demanda pra priorizar temas de YouTube longo
- Se NAO existir: flag interna `SEM_PESQUISA_YT = true` — temas YouTube serao marcados

**Material ja gravado (deduplicacao):**
- Busque `{{sessoes}}/*workshop*` e `{{sessoes}}/*download*` — historias e conteudo ja gravado
- Leia os titulos e temas principais de cada sessao encontrada
- Monte lista interna `TEMAS_JA_GRAVADOS[]` com: titulo do tema + sessao de origem
- Essa lista sera usada no Passo 3C pra evitar sugerir gravacao de algo que ja existe

## Passo 2 — Extrair os 3 insumos do cruzamento

### REGRA DE FONTES (CRITICA)

TODA informacao usada nos cruzamentos DEVE vir de um arquivo que voce LEU nesta sessao.
- Tendencias: so do arquivo de radar lido no passo 1. NAO inventar tendencias.
- Dados de engagement: so dos arquivos de analise lidos. NAO inventar numeros.
- Se quiser adicionar algo que NAO esta nos arquivos (ex: Google Trends, noticia recente), marque como `[INFERIDO — verificar]` e explique de onde veio.
- NUNCA apresentar informacao inferida como se fosse do radar.

### A) Tendencias quentes (do radar)

Do radar LIDO NO PASSO 1, extraia uma lista compacta:

```
TENDENCIAS QUENTES:
1. [Tendencia] — apareceu em [N] perfis — engagement: [dados] — [por que esta quente]
   Fonte: {{analises}}/YYYY-MM-DD-radar-semanal.md
2. [Tendencia] — ...
```

Incluir:
- Topicos que apareceram em 2+ perfis (sinal forte)
- Virais inesperados (fora do padrao do perfil = tema com demanda latente)
- Gaps: topicos que NENHUM perfil cobriu mas tem demanda (olhar comentarios/perguntas)

**IMPORTANTE:** So listar tendencias que ESTAO NO ARQUIVO DO RADAR. Se o radar nao menciona um topico, ele NAO e tendencia confirmada.

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

### REGRA DE TERRITORIO (CRITICA)

Antes de gerar qualquer angulo, aplicar o filtro:
- **Fala COM empresario/gestor?** → angulo do Marco, NAO do {{criador}}. Descartar.
- **Mostra BASTIDOR de como pensa/constroi?** → angulo do {{criador}}. Manter.
- **Linguagem de dev (codar, deploy, programar)?** → reescrever como Arquiteto (pensar, orquestrar, construir).
- **Repacoteia angulo de concorrente?** → NAO usar. O {{criador}} precisa ter LENTE PROPRIA (opiniao, vivencia, dado pessoal).

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

**PRIORIZAR sessoes tipo `download` e `brainstorm`** — estas costumam ter HISTORIAS CATALOGADAS
(banco de conteudo com emocao, uso potencial, frase-chave). Procurar especificamente por:
- Tabelas ou listas de "historias" ou "banco de conteudo" dentro das sessoes
- Frases entre aspas que o {{criador}} falou (material autentico)
- Momentos de vulnerabilidade ou virada

Para cada sessao, extraia APENAS:
- Historias reais com resultado concreto ("fiz X, deu Y")
- Opinioes fortes com vivencia ("nao e teoria, eu vivo isso")
- Bastidores/vulnerabilidade que conectam

**PILAR PESSOAL (15%):** Garantir que pelo menos 3-4 pepitas sejam do pilar Pessoal
(jornada, bastidores, relacoes, humor). Se nao encontrar nas sessoes recentes,
buscar sessoes mais antigas (ate 60 dias) que tenham historias de vida catalogadas.

Cruzar com tendencias: "essa historia da um angulo forte pra essa tendencia?"

**CLASSIFICAR cada pepita como:**
- **AGORA** — tem timing do radar, acontecimento recente (<2 semanas), ou serie em andamento. Entra no cardapio.
- **BANCO** — historia forte sem timing. Fica na secao "banco" do cardapio. Nao forcar no mes so pra preencher pilar.
Pepitas AGORA > pepitas BANCO. Nunca forcear historia sem timing so porque o pilar esta fraco — buscar OUTRO angulo com timing.

### REGRA DE DEDUPLICACAO (CRITICA)

Antes de promover uma historia/pepita a sugestao de GRAVACAO NOVA:
1. Checar contra `TEMAS_JA_GRAVADOS[]` montado no Passo 1
2. Checar `{{analises}}/*cardapio*.md` por temas similares
3. Se o tema JA FOI GRAVADO em alguma sessao:
   - NAO sugerir como gravacao nova (desperdiça tempo)
   - Sugerir como `[CORTE EXISTENTE]`: "Fonte: [sessao]. Sugestao: extrair corte sobre [angulo especifico] do material ja gravado."
4. Se apareceu em cardapio anterior mas NAO foi gravado: pode reusar com angulo diferente

Exemplo: "9 anos de DJ" ja esta gravado em sessao de workshop. NAO sugerir ancora de 1h sobre isso. Sugerir corte especifico do material existente.

### 3D) Conhecimento relevante

Busque em `{{conhecimento}}/` por arquivos com tags que cruzem com pilares.
Leia apenas atualizados nos ultimos 60 dias.
Extrair dados, frameworks ou insights que ENRIQUECEM os angulos gerados.

### 3E) Aplicar padrao de hook (com VARIEDADE)

Para cada angulo gerado em 3A/3B/3C, escolher o padrao de hook que melhor funciona.

**REGRA DE VARIEDADE:** Maximo 2 angulos com o MESMO padrao de hook no cardapio inteiro.
Se ja usou "Provocacao com substancia" 2x, o proximo angulo DEVE usar outro padrao.
Distribuir entre todos os 7 padroes. Se perceber que esta repetindo a estrutura
"Todo mundo faz X. Ninguem faz Y." — PARE e force um padrao diferente (storytelling, humor, numero, vulnerabilidade).

```
ANGULO: "Engenharia de Contexto > Prompt Engineering"
  Hook: Provocacao com substancia
  Frase: "Prompt engineering ta morto. O cara que viraliza sabe disso."
  
ANGULO: "TDAH e vantagem (CEO Palantir) + meu HD de 1GB"
  Hook: Vulnerabilidade + sistema
  Frase: "CEO da Palantir diz que TDAH e vantagem. Concordo. Mas sem HD externo, o processador trava. Eu sei porque travo todo dia."
```

## Passo 4 — Apresentar angulos gerados

Apresente os angulos com CONTEXTO SUFICIENTE pro criador avaliar sem adivinhar.

Cada sugestao DEVE ter 5 campos obrigatorios alem do titulo:
- **Contexto:** 2-3 linhas explicando de onde veio a ideia, qual a tese central, por que e relevante agora
- **Fonte:** arquivo/radar/sessao especifico (caminho do arquivo)
- **Diferencial:** o que torna unico vs o que concorrentes ja fizeram sobre o tema
- **Cruzamento:** qual tendencia x qual lente x qual hook
- **CTA potencial:** Yabadoo lista espera | lead magnet via ManyChat | nenhum (identificar ANTES do cardapio quais angulos puxam CTA natural)

```
## Angulos gerados — [Periodo]

### Cruzamentos Radar x Lente (novos)
1. "[Frase-angulo]" — **[Pilar]**
   Contexto: [2-3 linhas: tese central + por que agora]
   Fonte: [arquivo especifico do radar/analise]
   Diferencial: [o que torna unico vs concorrentes]
   Cruzamento: Radar [X] x Lente #[N] [nome] | Hook: [padrao]
   
2. "[Frase-angulo]" — **[Pilar]**
   Contexto: [2-3 linhas]
   Fonte: [arquivo]
   Diferencial: [unico vs concorrentes]
   Cruzamento: Radar [X] x Lente #[N] | Hook: [padrao]

### ClickUp com timing (ideias brutas + radar)
1. "[Frase-angulo]" — **[Pilar]**
   Contexto: [2-3 linhas: ideia original + como o radar deu timing]
   Fonte: ClickUp [titulo] + [arquivo radar]
   Diferencial: [o que muda com o timing]
   Timing: [tendencia que ativou]

### Pepitas do cerebro (historias/vivencias)
1. "[Frase-angulo]" — **[Pilar]** — **[AGORA/BANCO]**
   Contexto: [2-3 linhas: qual historia, resultado concreto, emocao]
   Fonte: [sessao especifica — caminho do arquivo]
   Diferencial: [vivencia propria, nao teoria]
   Timing: [por que AGORA — ou "sem timing, banco"]
   Hook: [padrao]
   CTA potencial: [Yabadoo lista espera / lead magnet / nenhum]

### Cortes de material existente [CORTE EXISTENTE]
1. "[Frase-angulo]" — **[Pilar]**
   Fonte: [sessao ja gravada]
   Sugestao: [que trecho/angulo extrair do material]

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

#### YouTube (cenario A — solo, ~8min)

- **Tema:** angulo mais forte da semana, preparado com profundidade
- **Pilar principal** + fonte do cruzamento
- **Pontos pra gravar:** 5-7 pontos concretos
- **Output estimado:** 8 cortes editaveis
- Se `SEM_PESQUISA_YT = true`: marcar `[SEM PESQUISA YT — validar demanda antes de gravar]`

Se depende de timing e modo=mes:
```
GRAVACAO: RADAR — tema do momento
Tipo: YouTube
Pilar sugerido: [X] (definir quando rodar /social-radar da semana)
[SEM PESQUISA YT — validar demanda antes de gravar]
```

#### Ancora — 2 subtipos

**Tipo 1: Conversa {{criador}} + Marco** (padrao)

NAO e um tema unico. E uma PAUTA com 5-7 PONTOS DE DISCUSSAO variados.
Cada ponto cobre pilar potencialmente diferente e gera 2-3 cortes independentes.
O valor esta na variedade — uma ancora rende cortes pra todos os pilares na mesma sessao.

Para cada ponto da pauta:
- **Angulo** (1 frase)
- **Contexto** (2 linhas: de onde veio, tese, por que agora)
- **Pilar**
- **Cortes esperados:** 2-3

Output estimado total: ~14 cortes

**Tipo 2: Solo livre**

{{criador}} sozinho construindo ou falando. Formato tipo live, tema unico mais profundo.
Tambem serve pra demos `[DEMO]` de coisas construidas ao vivo.

- **Tema:** angulo forte do cruzamento
- **Pilar** + fonte
- **Pontos pra gravar:** 5-7 pontos concretos
- **Output estimado:** ~14 cortes

**Visita/convidado:** NAO entra no sugerir. Quando acontecer, entra como extra organico no banco de cortes.

→ **VALIDACAO DE PILARES (1/4):** Checar distribuicao de pilares nas gravacoes do mes.
Se algum pilar esta ausente nas gravacoes, considerar incluir ponto na pauta da ancora.

### B) Reels produzidos (3 por semana) — cenario A, {{editor}} edita

**REGRA: Reels custam producao (cenario A, edicao do {{editor}}).**
Investir producao APENAS no que JA SABE que funciona:
- Angulos com ER alto CONFIRMADO em analises de concorrentes
- Temas que BOMBARAM no TikTok como teste (regra: TikTok testa → Reel produz)
- Os cruzamentos MAIS FORTES do cardapio (top 3 de qualidade)

Para cada Reel:
- **Angulo** (do cruzamento — o MELHOR, nao o resto)
- **Pilar** + fonte
- **Formato:** Screencast [DEMO] | Talking head | Storytelling | Demo ao vivo
- **Contexto:** 2-3 linhas de por que esse angulo merece investimento de producao
- **Justificativa de producao:** por que investir edicao nesse (ER comprovado? Tema testado? Top-3?)
- Tag `[DEMO]` quando for demonstracao de algo construido

→ **VALIDACAO DE PILARES (2/4):** Checar % cumulativo (gravacoes + Reels). Desvio > 5% da meta? Rebalancear.

### C) Carrosseis (3 por semana)

{{criador}} produz com Claude Code (`/social-carrossel`).
Para cada carrossel:
- **Angulo** (do cruzamento — nao tema generico)
- **Pilar** + fonte
- **Contexto:** 2 linhas de tese central
- **Tipo de hook** que vai usar

→ **VALIDACAO DE PILARES (3/4):** Checar % cumulativo (gravacoes + Reels + carrosseis). Desvio > 5%? Rebalancear.

### D) TikTok nativo (5-7 por semana) — LABORATORIO

TikTok e lab de R$0. Aqui entram os EXPERIMENTAIS, nao os melhores angulos.
Grava e posta cru. Sem edicao. Se bombar → vira Reel produzido na semana seguinte.

Para cada TikTok:
- **Angulo** (pode ser versao crua de um cruzamento, provocacao rapida, teste de tema)
- **Pilar** + contexto (por que agora)
- Tag `[TESTE → REEL]` quando a intencao e testar antes de produzir Reel
- Tag `[DEMO]` quando for demo apontando celular pra tela

→ **VALIDACAO DE PILARES (4/4 — FINAL):** Checar % de TODAS as materias-primas do mes.
Se algum pilar esta >5% fora da meta (35/25/25/15): rebalancear ANTES de apresentar ao {{criador}}.
Prioridade de protecao: Pessoal (nunca abaixo de 10%) > TDAH > Visao > Arquiteto.

### O que NAO definir aqui

- **Cortes IG (11/semana):** emergem da gravacao → `/social-cortes`
- **YouTube Shorts:** repost dos melhores cortes
- **Hooks escritos e copy:** responsabilidade da `/social-copy`
- **Briefing pro {{editor}}:** responsabilidade da `/social-copy`

## Passo 6 — Apresentar e validar

Apresente o cardapio completo. Usar o template adequado conforme o tipo de gravacao:

```
## SEMANA N — DD/MM a DD/MM

### Gravacao — YouTube
Cenario: A — estudio solo
Angulo: "[Frase-angulo]"
Pilar: [X] | Cruzamento: [radar] x [lente #N]
[SEM PESQUISA YT — validar demanda antes de gravar] ← se aplicavel
Contexto: [2-3 linhas: tese, por que agora, diferencial]
Pontos pra gravar:
  1. [Ponto 1 — 1 frase]
  2. [Ponto 2 — 1 frase]
  3. [Ponto 3 — 1 frase]
  4. [Ponto 4 — 1 frase]
  5. [Ponto 5 — 1 frase]
  6. [Ponto 6 — 1 frase]
  7. [Ponto 7 — 1 frase]
Output estimado: 8 cortes editaveis

### Gravacao — Ancora (Conversa com Marco)
Cenario: B — mesa/conversa
Pauta de pontos:
  1. "[Ponto]" — Pilar: [X]
     Contexto: [2 linhas: de onde veio, tese, relevancia]
     Fonte: [arquivo especifico]
     Cortes esperados: [N]
  2. "[Ponto]" — Pilar: [X]
     Contexto: [2 linhas]
     Fonte: [arquivo]
     Cortes esperados: [N]
  3. "[Ponto]" — Pilar: [X]
     Contexto: [2 linhas]
     Fonte: [arquivo]
     Cortes esperados: [N]
  4. "[Ponto]" — Pilar: [X]
     Contexto: [2 linhas]
     Fonte: [arquivo]
     Cortes esperados: [N]
  5. "[Ponto]" — Pilar: [X]
     Contexto: [2 linhas]
     Fonte: [arquivo]
     Cortes esperados: [N]
Output estimado total: ~14 cortes
Distribuicao de pilares na pauta: Arq [N] / Visao [N] / TDAH [N] / Pessoal [N]

### Reels produzidos (3) — cenario A, {{editor}} edita

1. "[angulo]" — **[Pilar]** — [formato: Screencast/Demo/Talking head/Storytelling]
   Contexto: [2 linhas: tese + por que agora]
   Justificativa de producao: [por que investir edicao — ER comprovado? Tema testado? Top-3?]
   Cruzamento: [origem]
2. "[angulo]" — **[Pilar]** — [formato] [DEMO] ← se aplicavel
   Contexto: [2 linhas]
   Justificativa de producao: [motivo]
   Cruzamento: [origem]
3. "[angulo]" — **[Pilar]** — [formato]
   Contexto: [2 linhas]
   Justificativa de producao: [motivo]
   Cruzamento: [origem]

### Carrosseis (3)

1. "[angulo]" — **[Pilar]** — Hook: [tipo]
   Contexto: [2 linhas: tese central]
   Cruzamento: [origem]
2. "[angulo]" — **[Pilar]** — Hook: [tipo]
   Contexto: [2 linhas]
   Cruzamento: [origem]
3. "[angulo]" — **[Pilar]** — Hook: [tipo]
   Contexto: [2 linhas]
   Cruzamento: [origem]

### TikTok nativo (5-7) — LABORATORIO

1. "[angulo]" — **[Pilar]** | Por que agora: [contexto] [TESTE → REEL] ← se aplicavel
2. "[angulo]" — **[Pilar]** | Por que agora: [contexto]
3. "[angulo]" — **[Pilar]** | Por que agora: [RADAR — tema do momento]
4. "[angulo]" — **[Pilar]** | Por que agora: [contexto] [DEMO] ← se aplicavel
5. "[angulo]" — **[Pilar]** | Por que agora: [contexto]

### Distribuicao de pilares (esta semana)
Arq: [N] ([%]) | Visao: [N] ([%]) | TDAH: [N] ([%]) | Pessoal: [N] ([%])
Meta: 35/25/25/15 | Desvio: [OK / rebalancear X]

### Check
- [ ] Gravacao com angulo de cruzamento (nao tema generico)?
- [ ] Ancora com pauta multi-ponto (nao tema unico)?
- [ ] Reels usam APENAS angulos validados/fortes (justificativa de producao)?
- [ ] TikTok recebe experimentais (nao os melhores angulos)?
- [ ] 3 carrosseis com angulos distintos?
- [ ] 5-7 TikTok nativos?
- [ ] Pilares validados progressivamente e dentro de +-5%? Pessoal >= 10%?
- [ ] Nenhum tema duplica material ja gravado?
- [ ] Temas YouTube marcados se sem pesquisa de demanda?
- [ ] Cada item tem contexto + fonte + diferencial?
- [ ] Nenhum tema repetido de cardapios anteriores?
- [ ] Pelo menos 50% dos angulos sao NOVOS (nao existiam no backlog)?
- [ ] Minimo 3 screencasts/demos no mes (Reels [DEMO] + TikToks [DEMO])? Builder mostrando tela e o gap #1 do nicho.
- [ ] Angulos com potencial CTA Yabadoo identificados? (lista espera, lead magnet, ManyChat)
- [ ] Nenhum angulo "fala COM empresario"? (territorio do Marco, nao do {{criador}})

### NAO inclui (vem depois)
- Cortes IG (11/sem) → /social-cortes apos transcricao
- Hooks escritos e copy → /social-copy
- Briefing pro {{editor}} → /social-copy
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
   - `weeks[].recording`:
     - Se YouTube: `type: "youtube"`, `theme`, `pillar`, `source`, `talkingPoints` (array de strings), `estimatedCuts: 8`, `status: "suggested"`
     - Se Ancora conversa: `type: "ancora"`, `subtype: "conversa_marco"`, `talkingPoints` (array de objetos `{point, pillar, context, expectedCuts}`), `estimatedCuts: 14`, `status: "suggested"`
     - Se Ancora solo: `type: "ancora"`, `subtype: "solo_livre"`, `theme`, `pillar`, `talkingPoints` (array de strings), `status: "suggested"`
   - `weeks[].reels[]` — theme, pillar, format, context, productionJustification, tag (DEMO se aplicavel), status **"suggested"** (hook fica VAZIO — `/social-copy` preenche)
   - `weeks[].carousels[]` — theme, pillar, source, context, status **"suggested"** (hook fica VAZIO)
   - `weeks[].tiktok[]` — theme, pillar, context, tag (TESTE_REEL/DEMO se aplicavel), status **"suggested"**
   - `weeks[].slots[]` — NAO preencher (cortes vem do `/social-cortes`)
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
