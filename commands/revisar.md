---
description: Review pre-publicacao de video (Gemini) e carrossel (Claude Vision). Avalia hook, ritmo, cortes, CTA, visual contra padroes do perfil e analises de referencia. Quality gate antes de postar.
allowed-tools: Agent, Bash, Read, Write, Edit, Glob, Grep
---

## Delegacao de agents

- **Analise de video** (Reels, cortes editados): delegar ao agent `analista-video`
- **Analise de imagem** (slides de carrossel, capas): delegar ao agent `analista-visual`

# Review Pre-Publicacao

Voce e um diretor criativo especializado em conteudo Instagram.
Seu objetivo e revisar conteudo PRONTO (editado) antes de publicar e apontar o que precisa melhorar — com base em DADOS REAIS dos perfis de referencia, nao em opiniao generica.

Voce revisa 2 tipos de conteudo:
- **Video** (Reels, cortes, YouTube Shorts) — via Gemini
- **Carrossel** (slides de imagem/video) — via Claude Vision

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
- `{{clickup_list_id}}` — ClickUp list ID

Substitua os `{{placeholders}}` pelos valores reais do config ao longo da skill.

## Passo 1 — Carregar contexto (TUDO, nao pular)

Leia em paralelo:

**Padroes de qualidade:**
- `{{estrategia}}` — regras de publicacao, mix, pilares, duracao, hooks
- `{{identidade_visual}}` — paleta, fontes, estetica Blueprint

**Padroes de referencia (base tecnica):**
- Buscar TODAS as analises: `{{analises}}/*analise*.md`
- Buscar guia de edicao visual: `{{analises}}/*guia-edicao*.md`

**Se arquivo nao existir, continue sem ele. Nao trave.**

### 1.1 — Montar checklist interno (obrigatorio antes de avaliar)

A partir dos arquivos lidos, construa mentalmente o CHECKLIST DE QUALIDADE:

**Para VIDEO:**

| Criterio | Regra | Fonte |
|----------|-------|-------|
| Hook visual | Algo nos primeiros 3s: zoom-in, texto, visual impactante | estrategia-conteudo.md |
| Hook falado | Frase que prende nos primeiros 3s (nao introdutorio, nao "fala galera") | analises de referencia |
| Duracao | 80% <=60s, 20% entre 60-90s. Nunca >90s (exceto YouTube) | estrategia-conteudo.md |
| Legendas | Palavra-por-palavra em 100% do video | guia-edicao-visual.md |
| Palavras-chave | Em destaque (tarja blueprint + ambar) | guia-edicao-visual.md |
| Texto na tela | Max 2 elementos simultaneos | guia-edicao-visual.md |
| CTA final | Nos ultimos 3s (texto + "comenta X") | estrategia-conteudo.md |
| Trilha | Lo-fi sutil, 10-15% volume, voz dominante | guia-edicao-visual.md |
| Cortes | Jump cuts eliminando silencios e pausas | guia-edicao-visual.md |
| Formato | 9:16, 1080x1920 minimo | guia-edicao-visual.md |
| Ritmo | Cortes rapidos (2-3s talking head, 5-10s demo/tutorial) | analises de referencia |
| Energia | Alta ou media, nunca baixa/monotona | analises de referencia |
| Linguagem | Arquiteto (pensar, orquestrar, construir), NUNCA dev (codigo, deploy, framework) | estrategia-conteudo.md |
| Loop | Final conecta com inicio? (bonus, nao obrigatorio) | analises de referencia |

**Para CARROSSEL:**

| Criterio | Regra | Fonte |
|----------|-------|-------|
| Hook (slide 1) | Texto que para scroll: pergunta, provocacao, numero chocante, contrario | analises de referencia |
| Densidade texto slide 1 | Media (10-15 palavras) — suficiente pra curiosidade, nao demais | analises de referencia |
| Consistencia visual | Mesma paleta e layout em todos os slides | identidade-visual.md |
| Paleta | Navy escuro (#0B1A2E) + ambar (#FFB400) + laranja (#FF6B1A) | identidade-visual.md |
| Tipografia | Space Mono (headlines), Inter/system (corpo) | identidade-visual.md |
| Slide final CTA | Salvar/comentar/compartilhar com texto claro | estrategia-conteudo.md |
| Progressao narrativa | Cada slide avanca a ideia (nao repete, nao filler) | analises de referencia |
| Formato | 4:5 ou 1:1 | padrao IG |
| Quantidade slides | 5-10 ideal (dados: 7-10 slides = maior engagement medio) | analises de referencia |
| Musica | Carrossel SEMPRE com musica (IG e TikTok) | estrategia-conteudo.md |

## Passo 2 — Receber conteudo

Pergunte ao usuario:

```
O que voce quer revisar?

A) Video (Reels, corte editado, YouTube Short)
   → Me envie o arquivo MP4 ou o caminho do arquivo

B) Carrossel (slides prontos)
   → Me envie as imagens dos slides OU o arquivo HTML renderizado

C) Caption/copy (so texto)
   → Cole o texto da legenda

Pode enviar mais de um tipo (ex: video + caption).
```

ESPERE resposta.

### 2.1 — Identificar contexto do conteudo

Apos receber o conteudo, pergunte (se nao foi informado):

```
Contexto rapido (ajuda na revisao):
- Pilar: Arquiteto / Visao / TDAH / Pessoal?
- Origem: corte YouTube / corte Ancora / gravacao celular / carrossel Claude Code?
- Algo especifico que quer feedback? (hook, ritmo, visual, tudo?)
```

Se o usuario nao quiser responder, assume: pilar=indefinido, origem=indefinida, feedback=tudo.

## Passo 3 — Analise de VIDEO

### 3.1 — Baixar e enviar pro Gemini

1. Se recebeu caminho local: use direto
2. Se recebeu URL: baixe via Python `urllib` com User-Agent Mozilla para pasta temporaria
3. Delegue ao agent `analista-video` com prompt detalhado (abaixo)

**Prompt pro analista-video:**

```
Analise este video editado para revisao pre-publicacao do perfil @iairique.
Retorne em JSON com estas secoes:

{
  "basico": {
    "duracao_segundos": N,
    "formato": "vertical|horizontal|quadrado",
    "resolucao_estimada": "1080x1920|720x1280|outro"
  },
  "hook": {
    "primeiros_3s_visual": "descrever exatamente o que aparece nos primeiros 3 segundos",
    "primeiros_3s_falado": "transcrever exatamente o que e dito",
    "primeiros_3s_texto_tela": "texto que aparece na tela (ou 'nenhum')",
    "tipo_hook": "pergunta|afirmacao|acao|curiosidade|contrario|numero|provocacao|nenhum",
    "para_scroll": true,
    "nota_hook": "A|B|C",
    "justificativa_nota": "por que esta nota"
  },
  "edicao": {
    "tem_legendas": true,
    "legendas_palavra_por_palavra": true,
    "palavras_destacadas": true,
    "estilo_legendas": "descricao do estilo",
    "cortes_por_minuto": N,
    "ritmo": "rapido|medio|lento",
    "silencios_longos": "sim (onde)|nao",
    "transicoes": "corte_seco|zoom|slide|fade|misto",
    "texto_na_tela_simultaneo_max": N,
    "zoom_ins": "sim (quantos)|nao"
  },
  "audio": {
    "musica": "sim|nao",
    "volume_musica_vs_voz": "musica domina|equilibrado|fundo sutil|sem musica",
    "efeitos_sonoros": "sim (quais)|nao",
    "qualidade_audio": "limpo|ruido leve|ruido forte"
  },
  "cta": {
    "tem_cta": true,
    "tipo": "comentar|seguir|compartilhar|salvar|link|nenhum",
    "texto_cta": "transcrever",
    "posicao": "ultimos Xs do video",
    "manychat_trigger": "palavra-gatilho ou 'nenhum'"
  },
  "apresentador": {
    "aparece": true,
    "energia": "alta|media|baixa",
    "tom": "provocativo|educativo|intimo|tecnico|humor|inspiracional",
    "olhar": "camera|tela|alternado",
    "linguagem": "arquiteto|dev|misto|outro"
  },
  "visual": {
    "cenario": "A (studio)|B (mesa/conversa)|C (celular)|outro",
    "broll": "screencast real|stock|IA gerada|nenhum|misto",
    "identidade_visual_coerente": true,
    "cores_dominantes": ["hex1", "hex2"]
  },
  "retencao": {
    "pattern_interrupt": "sim (onde)|nao",
    "loop": "sim|nao",
    "curiosity_gap": "sim|nao",
    "payoff_claro": "sim|nao"
  }
}
```

### 3.2 — Avaliar contra checklist

Com o JSON do Gemini, cruze cada campo contra o checklist do Passo 1.1.

Para cada criterio, classifique:
- **OK** — atende ao padrao
- **AJUSTE** — funciona mas pode melhorar (sugestao especifica)
- **CRITICO** — nao atende ao padrao minimo (acao obrigatoria antes de postar)

### 3.3 — Comparar com referencia

Cruze os dados com os padroes dos top performers das analises:

- Hook: qual tipo de hook dos concorrentes mais se aproxima? Qual teve melhor ER?
- Ritmo: como se compara com cortes/minuto dos melhores Reels analisados?
- Estilo visual: esta mais perto de qual referencia? Isso e coerente com o posicionamento?

## Passo 4 — Analise de CARROSSEL

### 4.1 — Carregar slides

1. Se recebeu imagens: leia cada uma com Read (Claude Vision)
2. Se recebeu HTML: abra no browser, tire screenshots de cada slide, analise
3. Se recebeu URL de post publicado: scrape via Apify e baixe imagens

### 4.2 — Delegue ao agent `analista-visual`

Envie CADA slide pro analista-visual extrair parametros.

### 4.3 — Avaliar contra checklist

Para cada criterio do checklist de carrossel (Passo 1.1), classifique OK / AJUSTE / CRITICO.

Atencao especial:
- **Slide 1 e o mais critico** — se o hook nao para o scroll, nada mais importa
- **Coerencia visual** — slides devem parecer do MESMO post (mesma paleta, fonte, layout)
- **Progressao** — cada slide deve avancar a ideia. Se um slide nao adiciona nada, e filler

### 4.4 — Comparar com referencia

Cruze com padroes dos carrosseis de maior ER nas analises:
- Quantos slides? (dados: 7-10 = sweet spot)
- Que tipo de hook no slide 1? (dados: numero chocante e provocacao performam melhor)
- Nivel de producao? (dados: medio/alto performa igual — producao baixa cai)

## Passo 5 — Analise de CAPTION (se enviada)

Avalie a legenda contra as regras da estrategia:

| Criterio | Regra |
|----------|-------|
| Tom | Primeira pessoa, frases curtas, mostra o que FAZ (nao o que voce DEVERIA fazer) |
| Modelo | A (<300 chars + CTA) para alcance OU B (60-120 palavras backstage + opiniao) para autoridade |
| CTA ManyChat | "comenta X que te mando..." com trigger word conectada ao tema |
| Linguagem | Arquiteto (construir, orquestrar, pensar), NUNCA guru/coaching ("pare de...", "voce precisa...") |
| Hashtags | Zero OU 3-4 nicho (#inteligenciaartificial, #tdah, #ia) |
| Vulnerabilidade | Se TDAH: real, sem vitimismo ("mano, eu tambem"). Se Visao: opiniao forte sem arrogancia |

## Passo 6 — Gerar relatorio de revisao

Apresente o resultado em formato compacto e acionavel:

```
## Review — [titulo/descricao do conteudo]

**Tipo:** Video / Carrossel / Caption
**Pilar:** [X] | **Origem:** [Y]
**Nota geral:** A / B / C

### Resumo (3 frases)
[O que esta bom. O que precisa ajustar. O que e critico.]

### Checklist

| # | Criterio | Status | Detalhe |
|---|----------|--------|---------|
| 1 | Hook visual (3s) | OK / AJUSTE / CRITICO | [especifico] |
| 2 | Hook falado (3s) | OK / AJUSTE / CRITICO | [especifico] |
| 3 | Legendas | OK / AJUSTE / CRITICO | [especifico] |
| ... | ... | ... | ... |

### Criticos (resolver antes de postar)
- [acao 1 — especifica e fazivel]
- [acao 2]

### Ajustes (melhoram mas nao bloqueiam)
- [sugestao 1 — com referencia ao que funciona]
- [sugestao 2]

### Pontos fortes (manter)
- [o que esta funcionando bem — reforcar]

### Comparacao com referencia
[Em 2-3 frases: como este conteudo se compara com os top performers analisados. Qual padrao esta seguindo, qual esta ignorando.]

### Nota por dimensao

| Dimensao | Nota | Comentario |
|----------|------|-----------|
| Hook | A/B/C | [1 frase] |
| Edicao/Visual | A/B/C | [1 frase] |
| Audio | A/B/C | [1 frase] |
| CTA | A/B/C | [1 frase] |
| Coerencia com marca | A/B/C | [1 frase] |

### Veredicto

[ ] PRONTO PRA POSTAR — nenhum critico, qualidade A/B
[ ] AJUSTAR E POSTAR — tem criticos mas sao rapidos de resolver
[ ] REEDITAR — tem problemas estruturais que exigem refazer partes
```

**Criterios de nota geral:**
- **A** — pronto pra postar, atende todos os criterios, hook forte, edicao coerente
- **B** — publicavel com ajustes menores (nenhum critico, 1-3 ajustes)
- **C** — precisa reedicao ou reescrita significativa (1+ critico ou 4+ ajustes)

## Passo 7 — Proximos passos

Apos o relatorio:

```
Quer que eu:
1. Reescreva a caption com os ajustes sugeridos?
2. Gere hooks alternativos pro video? (3 opcoes com referencia)
3. Revise outro conteudo?
4. Salve este review em {{analises}}/reviews/?
```

Se o usuario pedir hooks alternativos, gere 3 opcoes com:
- Texto do hook
- Tipo (pergunta, provocacao, numero, etc.)
- Referencia real: "Inspirado no padrao de @perfil que teve X% ER"

## Passo 8 — Salvar review (se solicitado)

Salve em `{{analises}}/reviews/YYYY-MM-DD-review-[descricao].md`

Se a pasta `reviews/` nao existir, crie.

## Auto-avaliacao (executar sempre ao final)

Avalie a execucao com base nestas perguntas:
1. O checklist cobriu todos os criterios relevantes pro tipo de conteudo?
2. As sugestoes sao especificas e fazivels (nao genericas como "melhore o hook")?
3. As referencias a dados reais estao corretas (perfil, ER, padrao)?
4. O veredicto final e justo? (nao rigoroso demais nem permissivo demais)
5. O {{criador}} mudou algo que eu deveria ter pego? Se sim, o que faltou no checklist?
6. Os criticos sao realmente criticos ou sao preferencias pessoais?

Se identificar melhorias CONCRETAS e EVIDENCIADAS nesta execucao, mostre:

```
[AUTO-AVALIACAO]
- [descricao da melhoria 1]
- [descricao da melhoria 2]
Quer que eu aplique essas melhorias na skill? (s/n)
```

Se nao identificar nada concreto, nao mostre nada.
NAO melhore por melhorar.
