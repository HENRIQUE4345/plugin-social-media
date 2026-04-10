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

## Passo 0 — Carregar perfil e avaliar rampa de tempo

### 0.1 — Perfil

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

### 0.2 — Rampa de tempo (recomendacao de modo)

Antes de aceitar o modo declarado pelo criador, avaliar se ele faz sentido pro momento do mes.

**Leia:** `{{pasta_projeto}}/linha-de-producao-conteudo.md` — secao "Rampa" (ou equivalente). Se existir rampa documentada por semana, usa ela. Se nao existir, usa regra default: mes = 4 semanas de volume cheio; semana = 1 semana isolada.

**Calcule:**
- Data atual (hoje)
- Ultimo dia util do mes corrente
- Quantas semanas ainda cabem inteiras ate o fim do mes (contando so semanas com 5+ dias uteis restantes)

**Regra de recomendacao:**

| Semanas restantes no mes | Modo declarado | Recomendacao |
|---|---|---|
| 3+ | mes | Seguir — mes faz sentido |
| 3+ | semana | Seguir — usuario sabe o que quer |
| 2 | mes | **Sugerir trocar pra semana** — "so sobram 2 semanas uteis do mes, planejar 4 semanas vai gerar cardapio que nao vai executar. Quer rodar modo semana em vez de mes?" ESPERE resposta. |
| 1 ou menos | mes | **Forcar semana** — "so sobra 1 semana util no mes. Vou rodar modo semana." Nao perguntar, so avisar. |
| qualquer | semana | Seguir |

Se a rampa tem volume especifico pra semana atual (ex: "Semana 3 (21-26): ~12 posts IG + 5 TikTok"), USAR esse volume em vez do default (14 IG + 5 TikTok). Registrar na memoria de trabalho como `VOLUME_AJUSTADO_POR_RAMPA`.

Esse ajuste propaga pros Passos 5 e 6 — o Passo 5 usa `VOLUME_AJUSTADO_POR_RAMPA` em vez dos volumes fixos do mix padrao.

## Passo 1 — Carregar contexto (TUDO em paralelo)

Leia estes arquivos simultaneamente:

**Estrutura (o esqueleto):**
- `{{estrategia}}` — pilares, mix V2, flywheel, slots, banco de cortes

**Radar (o que esta quente):**
- Busque o radar mais recente: `{{analises}}/*radar*.md`
- Se NAO existir radar E modo=semana: avise "Sem radar. Recomendo rodar /social-radar antes." Pergunte se quer continuar. ESPERE resposta.
- Se NAO existir radar E modo=mes: tudo bem — marcar slots de timing

**Camada VISAO do criador — 4 fontes paralelas:**

1. **42 lentes/topicos:** `{{topicos}}` (ou fallback em `arquivo/topicos-conteudo-henrique.md` se marcado como "arquivado — cerebro e a fonte viva") — cada topico e uma LENTE com opiniao propria
2. **Playbook do criador:** busque via Glob `pique/playbooks/playbook-{{criador}}.md` ou `pique/playbooks/*playbook*{{perfil_sem_@}}*.md` — principios anti-TDAH, autoconhecimento, frameworks de trabalho
3. **Contexto consolidado (bloco de negocio):** `{{contexto}}` — PRESTAR ATENCAO em secoes chamadas "Como [produto] Aparece no Conteudo", "Nota sobre conteudo como motor", "Estrutura de Negocio". Sao os padroes literais de como Yabadoo/Pique aparecem (ex: "bastidor", "segundo cerebro que construi pra mim")
4. **Banco de historias nas sessoes:** busque `{{sessoes}}/*download*.md` e `{{sessoes}}/*workshop*.md`. Para cada arquivo, abrir e buscar DENTRO por:
   - Heading contendo "Banco de Conteudo", "Historias", "PARTE 5", "Banco de Historias", "Momentos-Chave"
   - Tabelas com colunas "Historia" + "Emocao" (padrao canonico)
   - Se nao achar nenhum desses, NAO ler o arquivo inteiro — marca como "sem banco catalogado"

**Padroes de hook (o que funciona):**
- `{{analises}}/*analise*.md` — extrair padroes de engagement com numero (ER, eng medio, frequencia nos top posts)
- `{{analises}}/mapa-posicionamento.md` — gaps e espacos vazios

**Noticias ja curadas (alimenta TEMA sem re-scrapar):**
- **Fonte A:** Drive `G:/Drives compartilhados/Pique Digital/Pique Digital/Pique News/*-pique-news.html` — pegar os 7 mais recentes
- **Fonte B (fallback):** WebFetch em `https://docs.pique.digital/publico/pique/news/?json`
- Extrair o bloco `<!-- PIQUE-NEWS-METADATA ... -->` de cada — nao parsear HTML inteiro
- Se nenhuma das fontes funcionar, marcar "Pique News indisponivel — TEMA depende so do radar"

**Posicionamento:**
- `{{contexto}}` — identidade, tom, publico (ja listado acima tambem no bloco de negocio — leitura unica)

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

### B) Universo VISAO do criador (triplo)

Em vez de tratar "42 topicos" como lista plana, montar TRES listas separadas — cada tipo e usado pra cruzamento diferente:

#### B.1 — `FRAMEWORKS_PROPRIOS[]` (tese estrutural)

Conceitos/sistemas/principios que o criador articulou como seus. Cada um tem nome, descricao curta, fonte, trecho original.

**Fontes a consolidar (ler dos arquivos do Passo 1):**
- Playbook do criador (principios anti-TDAH, autoconhecimento, gatilhos)
- `{{contexto}}` (Arquiteto de Contexto, conceitos de identidade)
- Sessoes tipo `download` recentes (frameworks que emergiram)
- Yabadoo/Pique (conceitos do produto que sao bandeiras do criador)

**Estrutura esperada por item:**
```
- nome: "Engenharia de Contexto"
  descricao: "IA e dados + contexto. Contexto explica dados. Prompt bonito nao salva."
  fonte: arquivo/topicos-conteudo-henrique.md linha 18
  trecho: "Prompt bonito nao salva. O que muda o jogo e MAPEAR e ESTRUTURAR contexto..."
```

Meta: consolidar 4-8 frameworks. Se achar mais, listar os mais fortes primeiro. Se achar menos de 3, avisar no Passo 4.

#### B.2 — `HISTORIAS_CATALOGADAS[]` (vivencia com emocao)

Historias reais com resultado concreto, vividas pelo criador. Saem das tabelas canonicas "Banco de Conteudo" das sessoes.

**Estrutura por item:**
```
- id: "DJ virada 2017"
  historia: "Show terminando 6h da manha, virada de chave 'nao quero mais isso'"
  emocao: "Coragem, destino"
  uso_potencial: "Video 'o dia que abandonei a musica'"
  fonte: sessoes/2026-03-12-2300-download-jornada-completa.md linha 322
  trecho: "[copy-paste literal da linha da tabela]"
```

Meta: 10-15 historias. Se encontrar menos, nao forca — e OK ter menos.

#### B.3 — `LENTES_42[]` (opiniao-topico)

Os 42 topicos de `{{topicos}}` — cada um e uma opiniao forte sobre um tema especifico.

**Estrutura por item:**
```
- id: #2
  titulo: "Engenharia de Contexto, nao Prompt Engineering"
  opiniao: "[resumo 1 linha]"
  fonte: arquivo/topicos-conteudo-henrique.md
```

Carregar TODOS os 42. Sao leves, so titulo + opiniao resumida.

---

**Como cada lista e usada no Passo 3:**

| Cruzamento | Usa | Resulta em |
|---|---|---|
| Radar x Framework | `FRAMEWORKS_PROPRIOS[]` | Angulo de tese estrutural ("explicar mundo com o framework X") |
| Radar x Historia | `HISTORIAS_CATALOGADAS[]` | Angulo de storytelling 1a pessoa ("aquele dia que vivi X conecta com esse tema") |
| Radar x Lente | `LENTES_42[]` | Angulo de opiniao forte ("tema quente sob a lente #N") |
| Historia x Tema de negocio | `HISTORIAS_CATALOGADAS[]` | Angulo autentico com mensagem de negocio embutida como bastidor |

### C) Formulas empiricas (hooks com numero real)

Em vez de lista hardcoded, a skill EXTRAI as formulas das analises concorrentes lidas no Passo 1. So formulas que voce pode apontar pra um POST ESPECIFICO com metrica sao consideradas validas.

Veja o Passo 2E pra extracao. A lista consolidada vai em `FORMULAS_EMPIRICAS[]`:

```
- nome: "Cultura pop reimaginada por IA"
  frequencia: "7/20 posts top (35%)"
  performance: "52k engagement medio (vs 11k geral)"
  estrutura: "Video IA + headline provocativa + personagem famoso"
  post_referencia: "Pixar em 3h = 72k+ likes"
  fonte: {{analises}}/2026-03-29-analise-diego-paulo.md
```

**Se nenhuma formula for extraida:** usar FALLBACK hardcoded (abaixo) e marcar cada item como `[INFERIDO]`:

```
FALLBACK — formulas nao validadas empiricamente nesta sessao:
- Numero chocante
- Provocacao com substancia
- Medo + solucao pratica
- Comparativo honesto
- Cultura pop reimaginada
- Tutorial implicito
- Vulnerabilidade + sistema
```

### D) Extrair frameworks, historias e lentes (passo operacional)

Este passo so roda se os arquivos do Passo 1 foram carregados com sucesso. Saida: 3 listas internas (`FRAMEWORKS_PROPRIOS[]`, `HISTORIAS_CATALOGADAS[]`, `LENTES_42[]`).

**Frameworks — regras de extracao:**

Dos arquivos do playbook + contexto + yabadoo + sessoes-download, buscar por:
- Frases em destaque (titulos, `**negrito**`, citacoes entre aspas) que nomeiam um conceito
- Secoes com heading tipo "Framework", "Principio", "Conceito", "Modelo mental"
- Blocos onde o criador articula "X = Y" (definicoes proprias)

Candidatos validos devem ter NOME + definicao de 1-2 linhas. Descarta nomes genericos ("produtividade", "foco") — so conceitos NOMEADOS e DISTINTIVOS.

**Historias — regras de extracao:**

Dos arquivos de sessao encontrados no Passo 1:
1. Se a sessao tem tabela com colunas "Historia" + "Emocao": extrair linha por linha, cada linha vira um item de `HISTORIAS_CATALOGADAS[]`
2. Se a sessao tem heading tipo "PARTE 5 — Historias", "Banco de Conteudo", "Historias e Momentos-Chave": ler a secao e extrair cada item listado
3. Se nao tem nenhum dos padroes: pular o arquivo

NAO ler sessoes inteiras em prosa procurando por narrativas — e custoso e impreciso. So tabelas/listas explicitas.

**Lentes — regras de extracao:**

De `{{topicos}}` (ou fallback em `arquivo/topicos-conteudo-henrique.md`):
- Cada heading `### N. "Titulo"` vira um item
- Extrair numero, titulo, primeira linha do corpo (resumo da opiniao)
- Carregar todos os 42 (ou quantos tiver)

### E) Extrair formulas empiricas (passo operacional)

Dos arquivos `{{analises}}/*analise*.md` do Passo 1:

Para cada arquivo de analise, buscar:
- Tabelas com colunas "Formula", "Padrao", "Frequencia", "ER", "engagement"
- Secoes com heading tipo "Formula #N", "Padrao validado", "Top posts"
- Blocos numerados que descrevem padroes de conteudo com METRICA

Cada candidato precisa ter OBRIGATORIAMENTE:
- Nome do padrao
- Metrica (frequencia nos top OU ER OU engagement medio)
- Post de referencia (exemplo concreto que a analise cita)

Se faltar metrica, NAO incluir (vira ruido).

Consolidar em `FORMULAS_EMPIRICAS[]`. Meta: 5-8 formulas. Se menos de 3, usar fallback hardcoded marcado como `[INFERIDO]`.

## Passo 3 — CRUZAMENTO CRIATIVO (o core da skill)

Este e o passo que gera valor. NAO pule, NAO simplifique.

### REGRA DE TERRITORIO (CRITICA)

Antes de gerar qualquer angulo, aplicar o filtro:
- **Fala COM empresario/gestor?** → angulo do Marco, NAO do {{criador}}. Descartar.
- **Mostra BASTIDOR de como pensa/constroi?** → angulo do {{criador}}. Manter.
- **Linguagem de dev (codar, deploy, programar)?** → reescrever como Arquiteto (pensar, orquestrar, construir).
- **Repacoteia angulo de concorrente?** → NAO usar. O {{criador}} precisa ter LENTE PROPRIA (opiniao, vivencia, dado pessoal).

### 3A) Radar x VISAO — angulos novos

Para CADA tendencia quente, cruzar contra OS TRES TIPOS de VISAO (framework, historia, lente). Cada tipo gera um angulo com perfil diferente.

```
TENDENCIA: [Ex: "Claude Code dominando 4/10 perfis"]

  x FRAMEWORK "Engenharia de Contexto":
    "Todo mundo postando sobre Claude Code. Ninguem explicou que
     o segredo nao e a ferramenta — e o CONTEXTO que voce da."
    → tese estrutural — angulo explicativo

  x HISTORIA "Beco 185k lancamentos":
    "Rodei Claude Code em cima de 185 mil lancamentos financeiros
     do Beco. Achei 1.687 anomalias que ninguem viu. A ferramenta
     foi a mesma que todo mundo usa — mudou o contexto."
    → storytelling 1a pessoa — angulo com resultado real

  x LENTE #42 "IA e burra por sua culpa":
    "Ta digitando prompt no Claude Code? Por isso teu resultado
     e generico. Fala com ele — teclado e interface de 1870."
    → opiniao forte — angulo provocativo

  → 3 angulos candidatos, cada um com perfil distinto
  → aplicar filtro de negocio (Passo 3F) em cada um antes de finalizar
  → escolher 1-2 melhores por tendencia baseado em: variedade de formula empirica (Passo 3E), nivel de conexao com visao, encaixe natural com pilar
```

Fazer isso pra pelo menos as 5 tendencias mais quentes. Gerar 2-3 candidatos por tendencia, escolher os melhores. Antes de finalizar CADA angulo, passar pelo Passo 3F (filtro de negocio) pra atribuir `nivel_negocio` e `alvo`.

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

### 3E) Aplicar FORMULA EMPIRICA + escrever HOOK SEED

Para cada angulo gerado em 3A/3B/3C, escolher **uma das `FORMULAS_EMPIRICAS[]` extraidas no Passo 2E** (ou fallback se extracao falhou). Cada angulo fica amarrado a uma formula com numero real.

**REGRA DE VARIEDADE:** Maximo 2 angulos com a MESMA formula no cardapio inteiro. Se ja usou "Cultura pop reimaginada" 2x, o proximo angulo precisa outra. Distribuir entre as formulas disponiveis.

**O que cada angulo precisa ter apos este passo:**

```
ANGULO: "[frase-angulo]"
  formula_empirica:
    nome: "Storytelling 1a pessoa"
    performance: "ER 14.83% vs 4.16% terceira pessoa"
    post_referencia: "@thaismartan 'Melhor IA?' — 12.542 likes"
    fonte: {{analises}}/2026-03-29-analise-thaismartan-eusoutwins.md
  hook_seed: "[1 frase curta — ponto de partida pra /social-copy expandir. NAO e a copy final.]"
```

**Hook seed — regras:**
- 1 frase, maximo 15 palavras
- Captura a TESE do angulo em tom de headline
- Usa a linguagem do criador (do contexto + playbook)
- E um PONTO DE PARTIDA, nao a copy final. `/social-copy` pode expandir em 3-5 versoes

**Exemplo:**

```
ANGULO: "CEO da Palantir valida o que vivo todo dia"
  formula_empirica: Vulnerabilidade + sistema
  hook_seed: "CEO da Palantir disse que TDAH e vantagem. Concordo. Mas sem HD externo, o processador trava."
```

### 3F) Filtro de NEGOCIO (nao e skill de vendas)

Para cada angulo que passou pelos Passos 3A/3B/3C + 3E, atribuir `nivel_negocio` e `alvo`. Este passo NAO empurra Yabadoo/Pique em todo angulo — ele MEDE e SINALIZA.

**4 niveis possiveis:**

| Nivel | Descricao | Exemplo |
|---|---|---|
| `direta` | Produto e o tema do post | "Abri o beta do Yabadoo" |
| `indireta` | Tema generico mas o exemplo usa o produto | "Como organizo ideias com IA — mostra tela do Yabadoo" |
| `ambiente` | Tema livre, produto aparece no background/bastidor | "Peguei Gravei falando sobre build — tela aberta no Yabadoo" |
| `nenhuma` | Angulo puro de autoridade, sem mencao ao produto | "TDAH como superpoder" (sem amarrar ao produto) |

**Alvo:** `Yabadoo` | `Pique` | `nenhum`

**Regras de atribuicao (ordem):**

1. **Default e `nenhuma`.** So subir nivel se o encaixe for NATURAL — nao forcar.
2. **Consultar `{{contexto}}`** secao "Como [produto] Aparece no Conteudo" (ou equivalente). Se a linguagem do angulo encaixa com um dos padroes listados la, o nivel e o que o padrao pede. Ex: se o contexto diz "aparece como segundo cerebro", e o angulo e uma historia de TDAH, o nivel natural e `ambiente` (produto aparece no background quando o criador fala da propria rotina).
3. **Territorio vs alvo:** se o angulo e territorio `{{criador}}` (bastidor), alvo provavel e Yabadoo (Yabadoo e mais ligado ao criador individual). Se for territorio de outro perfil (fala com empresario), descartar antes desse passo (regra de territorio ja existente).
4. **Nivel `direta` e RARO.** So quando o tema do post literalmente e o produto (lancamento, beta, demo especifica). Se o angulo e "vou mostrar como uso Yabadoo", e `indireta`, nao `direta`.
5. **`como_aparece` — 1 frase concreta.** Nao vale "Yabadoo aparece no post". Precisa ser tipo "mencionado como 'o cerebro que construi pra mim' durante a historia da organizacao do TDAH".

**Saida deste passo — para cada angulo:**

```
ANGULO: "[frase]"
  camada_negocio:
    nivel: ambiente
    alvo: Yabadoo
    como_aparece: "tela aberta no Yabadoo enquanto conta a historia de organizar 185k lancamentos do Beco"
```

**Medicao final do passo:**

Contar a distribuicao ao final:
```
Distribuicao de negocio (N angulos totais):
  direta: X (X%)
  indireta: X (X%)
  ambiente: X (X%)
  nenhuma: X (X%)

Angulos com conexao >= indireta: X (X%)
```

**Sinalizacao (nao bloqueio):**

- Se `>= indireta` < 30%: avisar `"Atencao: apenas X% dos angulos tocam Yabadoo/Pique em algum nivel. Cardapio enviesado pra autoridade pura — e isso uma escolha consciente ou quer reforcar?"` e ESPERAR resposta do criador.
- Se `direta` > 25%: avisar `"Muitos angulos diretos ao produto (X%). Risco de soar venda. Quer rebalancear pra mais ambiente/indireta?"` e ESPERAR resposta.
- Caso contrario, seguir sem perguntar.

### 3G) Montar BRIEF DE HANDOFF

Para cada angulo aprovado ate aqui, definir `destino` (onde vai parar no pipeline de producao) + `hints` especificos por destino. Esse passo garante que `/social-copy` e `/social-carrossel` nao reinventam o angulo — ja recebem material pronto pra executar.

**Destinos possiveis:**

- `gravacao-youtube` — video longo solo
- `gravacao-ancora-ponto` — 1 ponto dentro da pauta da ancora (H+M ou solo)
- `reel-produzido` — Reel cenario A (Solo ou Dividida), editado pelo {{editor}}
- `carrossel` — carrossel imagem/video, gerado por `/social-carrossel`
- `tiktok-nativo` — Peguei Gravei cru no TikTok
- `corte-existente` — extrair de material ja gravado (ver regra de deduplicacao)

**Hints por destino:**

Pra cada angulo, preencher os hints relevantes pro destino escolhido:

| Destino | Hints obrigatorios |
|---|---|
| `carrossel` | `formato_narrativo` (problema→cenarios / lista / provocacao→evidencia / historia→framework / antes→depois), `slides_estimados` (5-8) |
| `reel-produzido` | `frase_candidata_headline` (literal, do hook_seed), `formato_reels` (solo / dividida / talking-head / screencast) |
| `tiktok-nativo` | `formato_tiktok` (peguei-gravei / build-celular / reacao), `urgencia` (hoje / semana / quando-der) |
| `gravacao-youtube` | `pontos_pra_gravar` (5-7 pontos concretos), `output_estimado` (8 cortes) |
| `gravacao-ancora-ponto` | `ponto_na_pauta` (numero), `cortes_esperados` (2-3) |
| `corte-existente` | `sessao_fonte`, `trecho_alvo`, `angulo_especifico` |

**Material de apoio (`material_apoio[]`):**

Lista de arquivos que o executor (`/social-copy` ou `/social-carrossel`) vai ler pra executar o brief. So caminhos ABSOLUTOS, so arquivos que alimentam DIRETAMENTE este angulo. Nao e lista generica.

Exemplo:
```
material_apoio:
  - sessoes/2026-03-12-2300-download-jornada-completa.md (historia fonte)
  - projetos/marca-iairique/analises/2026-03-29-analise-diego-paulo.md (formula empirica de referencia)
  - projetos/marca-iairique/contexto-iairique-consolidado.md linhas 71-77 (padrao de como Yabadoo aparece)
```

## Passo 4 — Apresentar briefs de angulo (schema de 7 blocos)

Apresente CADA angulo como um **brief estruturado** com 7 blocos. Este e o CONTRATO da skill — copy e carrossel vao consumir exatamente estes blocos e nao devem reinventar nada.

**Estrutura obrigatoria por angulo:**

```
## ANGULO [N]: "[frase-angulo de 1 linha — cristalizacao do cruzamento]"

**1. TESE**
[2-3 linhas — qual e a POSTURA do criador, o ponto de vista dele, o que ele defende. NAO e descricao do tema. E "o que o criador tem a dizer sobre isso".]

**2. CAMADA TEMA** (o que esta quente)
- origem: [radar | mapa-posicionamento | gap | noticia-pique-news | serie]
- referencia: "[trecho copy-paste do arquivo]" — [caminho do arquivo]
- por_que_agora: [1 linha de timing concreto]

**3. CAMADA VISAO** (lente do criador)
- tipo: [framework | historia | lente-topico | opiniao-forte | vivencia]
- id: [nome/identificador — ex: "Engenharia de Contexto" | "DJ virada 2017" | "Lente #42"]
- fonte: [caminho absoluto do arquivo]
- trecho: "[2-3 linhas copy-paste do material original]"

**4. CAMADA NEGOCIO** (mensagem — do Passo 3F)
- nivel: [direta | indireta | ambiente | nenhuma]
- alvo: [Yabadoo | Pique | nenhum]
- como_aparece: [1 linha concreta — ex: "menciona como 'o cerebro que construi pra mim' no meio da historia"]

**5. FORMULA EMPIRICA** (hook escolhido — do Passo 3E)
- nome: [nome da formula de `FORMULAS_EMPIRICAS[]`]
- performance: "[metrica real — ex: '35% dos top posts, 52k eng medio']"
- post_referencia: "[trecho + caminho]"
- hook_seed: "[1 frase curta, max 15 palavras — ponto de partida pra /social-copy expandir]"

**6. MATERIAL DE APOIO** (pro executor nao reler tudo)
- [caminho 1 — arquivo que alimenta este angulo especifico]
- [caminho 2]
- [caminho 3]

**7. HANDOFF** (destino e hints — do Passo 3G)
- destino: [gravacao-youtube | gravacao-ancora-ponto | reel-produzido | carrossel | tiktok-nativo | corte-existente]
- hints:
  [campos especificos por destino — ver tabela do Passo 3G]

**Pilar:** [Arquiteto | Visao | TDAH+Sistema | Pessoal]
```

**Regras de apresentacao:**

- Agrupar os angulos POR PILAR no output, nao por bucket/origem
- Contar quantos angulos foram gerados por cada tipo de cruzamento (Radar x Framework | Radar x Historia | Radar x Lente | ClickUp x Radar | Corte existente)
- Imprimir no topo a medicao do Passo 3F (distribuicao de negocio) e a distribuicao de pilares
- Se o cardapio tiver >20 angulos, apresentar em lotes de 5-10 com "continuar?" entre eles — e briefing denso, nao sobrecarregar

**Apos apresentar todos os angulos:**

```
---

## Resumo

Angulos totais: N
- Por pilar: Arq X | Visao X | TDAH X | Pessoal X
- Por tipo de cruzamento: R×F X | R×H X | R×L X | C×R X | Corte X
- Distribuicao negocio: direta X | indireta X | ambiente X | nenhuma X
- Conexao >= indireta: X%

Semana/mes alvo: [do Passo 0]
Volume da rampa: [se VOLUME_AJUSTADO_POR_RAMPA foi aplicado]

Proxima etapa: monte o cardapio (Passo 5) — distribui estes angulos nas materias-primas.
```

**ESPERE o {{criador}} validar antes de continuar pro Passo 5.** Ele pode:
- Aprovar tudo ("segue")
- Reprovar angulos especificos (remover)
- Ajustar nivel de negocio de angulos especificos
- Pedir mais cruzamentos de um tipo especifico
- Trocar a formula empirica de um angulo
- Reescrever a tese ou o hook_seed

**REGRA DE IMUTABILIDADE:** Uma vez que o criador aprovou um angulo, o brief e IMUTAVEL. Copy/carrossel nao podem mudar tese, camada de negocio, formula empirica ou hook_seed sem o criador explicitamente pedir reabertura.

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
2. Atualize o objeto com os dados do cardapio aprovado. Cada item de materia-prima carrega um **brief completo de 7 blocos** (contrato do Passo 4) em um sub-objeto `brief`. Os campos antigos (`theme`, `pillar`, `context`, `source`, `productionJustification`) sao MANTIDOS pra retrocompatibilidade com o dashboard HTML — eles viram derivados do brief.

**Schema de um item (reel/carrossel/tiktok/ponto-ancora) apos este passo:**

```json
{
  "theme": "[frase-angulo]",
  "pillar": "[pilar]",
  "format": "[formato quando aplicavel]",
  "context": "[tese — 2-3 linhas]",
  "source": "[caminho do arquivo fonte principal]",
  "productionJustification": "[por que investir edicao]",
  "tag": "[DEMO/TESTE_REEL/null]",
  "status": "suggested",
  "brief": {
    "angulo": "[frase-angulo]",
    "tese": "[2-3 linhas]",
    "tema": {
      "origem": "radar|mapa-posicionamento|gap|noticia|serie",
      "referencia": "[trecho]",
      "fonte": "[caminho]",
      "por_que_agora": "[1 linha]"
    },
    "visao": {
      "tipo": "framework|historia|lente-topico|opiniao-forte|vivencia",
      "id": "[nome]",
      "fonte": "[caminho]",
      "trecho": "[2-3 linhas]"
    },
    "negocio": {
      "nivel": "direta|indireta|ambiente|nenhuma",
      "alvo": "Yabadoo|Pique|nenhum",
      "como_aparece": "[1 linha]"
    },
    "formula_empirica": {
      "nome": "[nome]",
      "performance": "[metrica]",
      "post_referencia": "[trecho + caminho]",
      "hook_seed": "[1 frase]"
    },
    "material_apoio": ["[caminho 1]", "[caminho 2]"],
    "handoff": {
      "destino": "carrossel|reel-produzido|tiktok-nativo|gravacao-youtube|gravacao-ancora-ponto|corte-existente",
      "hints": { /* campos especificos por destino */ }
    }
  }
}
```

**Estrutura por categoria:**
   - `weeks[].recording`:
     - Se YouTube: `type: "youtube"`, `theme`, `pillar`, `source`, `talkingPoints[]`, `estimatedCuts: 8`, `status: "suggested"`, `brief: {...}` (o brief se aplica ao angulo mae da gravacao)
     - Se Ancora conversa: `type: "ancora"`, `subtype: "conversa_marco"`, `talkingPoints: [{point, pillar, context, expectedCuts, brief: {...}}]` — cada ponto da pauta carrega seu proprio brief
     - Se Ancora solo: `type: "ancora"`, `subtype: "solo_livre"`, `theme`, `pillar`, `talkingPoints[]`, `brief: {...}`
   - `weeks[].reels[]` — campos antigos + `brief`
   - `weeks[].carousels[]` — campos antigos + `brief`
   - `weeks[].tiktok[]` — campos antigos + `brief`
   - `weeks[].slots[]` — NAO preencher (cortes vem do `/social-cortes`)
   - `updatedAt` — data atual (YYYY-MM-DD)

3. Salve o JSON via Write (substituicao completa do arquivo)

**Retrocompatibilidade:**
- Itens existentes no JSON que NAO tem `brief` continuam validos — dashboard HTML ignora campo desconhecido, brief e opcional no schema
- Adicionar brief nos itens novos (aprovados nesta sessao), nao reescrever itens antigos

**Se NAO existir:**
Avise "JSON de content nao encontrado. Rode `/social:planejar` pra gerar os dados do mes."
Salve apenas o markdown como antes.

**IMPORTANTE:** Nao busque nem modifique nenhum arquivo HTML. Edite apenas o JSON.
**IMPORTANTE:** Campos de copy (title, hook, briefing) ficam vazios no nivel raiz — `/social:copy` preenche depois lendo `brief.hook_seed` como ponto de partida.

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
