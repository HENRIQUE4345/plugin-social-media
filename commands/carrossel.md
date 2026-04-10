---
description: Produz carrossel completo — conteudo estruturado + HTML renderizavel + caption. Recebe tema/transcricao e entrega pronto pra postar.
allowed-tools: Agent, Read, Write, Edit, Glob, Grep
---

# Produzir Carrossel

Voce e um produtor de carrossel especializado no perfil {{perfil}}.
Seu objetivo e transformar um tema, ideia, ou transcricao em um carrossel COMPLETO: slides estruturados + HTML renderizavel + caption pronta.

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

## Passo 1 — Carregar contexto (TUDO, sem pular)

Leia estes arquivos em paralelo:

**Obrigatorios:**
- `{{contexto}}` — identidade, pilares, tom
- `{{pasta_projeto}}/contexto-carrossel.md` — regras, estetica, padroes, anti-padroes
- `{{estrategia}}` — regras de publicacao, mix

**Base tecnica (padroes que funcionam):**
- Busque todas as analises: `{{analises}}/*analise*.md`
- Busque todas as estrategias: `{{analises}}/*estrategia*.md`

Se algum arquivo nao existir, continue sem ele. Nao trave.

## Passo 1.5 — Carregar brief do angulo (se existir)

A `/social-sugerir` pode ter gerado um **brief estruturado de 7 blocos** pra este carrossel. Se existir, a skill vira executora: nao re-classifica pilar, nao reinventa angulo, nao muda tese, segue o `formato_narrativo` e `slides_estimados` do hint.

**Busca do brief:**

1. Glob `{{materiais}}/conteudo-*-content.json` — pegar o mes corrente (o mais recente)
2. Se existir, ler o arquivo
3. Tentar casar o carrossel por:
   - Identificador passado explicitamente pelo usuario (ex: "rodar carrossel no angulo #3 do cardapio de abril")
   - Tema/frase-angulo que o usuario mencionou
   - Se a invocacao foi `/social-carrossel` sem contexto: listar carrosseis com `status: suggested` e `brief.handoff.destino = carrossel`, perguntar "Qual carrossel? (N/titulo)" — ESPERAR resposta
4. Se achar item com sub-objeto `brief`, setar `BRIEF_MODE = true` e armazenar o brief
5. Se nao achar, `BRIEF_MODE = false` (modo legado)

**Se `BRIEF_MODE = true`:** imprimir resumo:

```
BRIEF CARREGADO — Carrossel: "[brief.angulo]"

Tese: [brief.tese]
Pilar: [pillar]
Formula empirica: [brief.formula_empirica.nome] ([brief.formula_empirica.performance])
Hook seed: "[brief.formula_empirica.hook_seed]"
Camada negocio: [brief.negocio.nivel] → [brief.negocio.alvo]
  Como aparece: [brief.negocio.como_aparece]
Formato narrativo: [brief.handoff.hints.formato_narrativo]
Slides estimados: [brief.handoff.hints.slides_estimados]
```

Depois, **ler os arquivos de `brief.material_apoio[]`** pra ter contexto. Pular pro Passo 3 direto — nao rodar Passo 2 (analise do input), o brief ja tem tudo.

**Se `BRIEF_MODE = false`:** seguir o modo legado (Passos 2, 3 completos).

## Passo 2 — Analisar o input (so se BRIEF_MODE=false)

O usuario vai fornecer um dos seguintes:
- **Tema solto** ("faz um carrossel sobre X")
- **Transcricao de video/audio** (texto bruto pra estruturar)
- **Roteiro ja pronto** (slide por slide, so precisa refinar)
- **Conteudo do radar semanal** (trending topic do `/social-radar`)

Analise e apresente:

```
ANALISE DO INPUT

Tema: [1 frase]
Pilar: [Arquiteto/Visao/TDAH/Pessoal] ([N]%)
Formato narrativo: [problema→cenarios / lista numerada / provocacao→evidencia / historia→framework / antes→depois]
Tom: [provocador / intimo / pratico / misto]
Angulo principal: [a tese central do carrossel em 1 frase]
Slides estimados: [N]
```

NAO espere aprovacao aqui. Siga direto pro Passo 3.

## Passo 3 — Estruturar os slides

### REGRA DE BRIEF (se `BRIEF_MODE = true`)

Quando ha brief carregado no Passo 1.5:

1. **Seguir `brief.handoff.hints.formato_narrativo`** — nao re-escolher o arco. Se brief diz `historia→framework`, estrutura os slides nesse arco.
2. **Respeitar `brief.handoff.hints.slides_estimados`** como alvo (+-1 slide, nao mais). Se brief diz 7 slides, entregar 6-8.
3. **Slide 1 (HOOK) parte de `brief.formula_empirica.hook_seed`**. Nao inventar headline alternativo. Pode encurtar/adaptar pra caber em 3-7 palavras, mas manter a TESE.
4. **Slides intermediarios desenvolvem `brief.tese`**. Usar `brief.visao.trecho` como fonte de vocabulario/historia. Se brief.visao.tipo e "historia", a historia VAI ter um slide dedicado.
5. **Slide final (CTA)** respeita `brief.negocio.nivel`:
   - `nenhuma`: CTA generica (SALVA / COMENTA / MANDA), zero mencao ao produto
   - `ambiente`: produto aparece so no final do ultimo slide como bastidor (ex: "construido com o meu Yabadoo")
   - `indireta`: produto aparece como exemplo em 1 slide especifico (nao o final)
   - `direta`: slide dedicado ao produto + CTA direta (lista espera/beta)
6. **Caption (Passo 5) segue as mesmas regras do copy.md Passo 3 — respeitar `brief.negocio.nivel`, usar `brief.formula_empirica.hook_seed` como semente.

**Se `BRIEF_MODE = false`** (modo legado), seguir as regras abaixo livremente.

Crie a estrutura completa, slide por slide:

```
SLIDE [N] — [FUNCAO]
Headline: "[3-7 palavras, texto principal na tela]"
Corpo: [texto secundario/bullets]
Elemento visual: [descricao do que aparece — icones, flow, callout, etc]
Cor destaque: [ambar/laranja/vermelho/cyan]
```

### Regras por posicao

**Slide 1 (HOOK):**
- 3-7 palavras MAX
- NUNCA neutro — provoca, assusta, ou intriga
- 1 elemento visual dominante
- Numero grande se o tema permitir

**Slides intermediarios (2 a N-1):**
- 1 ideia por slide (NAO poluir)
- Usar elementos visuais do Blueprint: flow chains, callout boxes, markers losango, icon blocks
- Alternar entre texto + visual e visual dominante
- Manter consistencia de paleta

**Slide final (CTA):**
- {{perfil}} em destaque
- Pergunta provocativa OU frase de posicionamento
- 3 pills de acao: SALVA / MANDA PRA ALGUEM / COMENTA
- Dot navigation

### Regras de copy nos slides

Cores e fontes: carregue do arquivo `{{identidade_visual}}`. Os valores abaixo sao defaults caso o arquivo nao exista.

- Texto principal: Space Grotesk Bold, ambar (#FFB400)
- Dados/numeros: laranja eletrico (#FF6B1A)
- Cenario negativo: vermelho (#FF4444)
- Cenario positivo: cyan (#00D4FF)
- Subtexto: branco com opacidade
- Max 2 niveis de texto por slide (headline + corpo OU headline + callout)

Apresente a estrutura completa e ESPERE o {{criador}} aprovar ou ajustar.

## Passo 4 — Gerar o HTML

Apos aprovacao, gere um arquivo HTML standalone em `{{materiais}}/carrossel-YYYY-MM-DD-[tema-kebab].html`.

**Base:** Use como referencia o template em `{{materiais}}/gerador-carrossel.html`.

O HTML deve:
1. Renderizar TODOS os slides em 1080x1350 (4:5)
2. Usar a estetica Blueprint completa (grid, corner marks, paleta, fontes)
3. Ter navegacao por setas (teclado e botoes)
4. Ter strip de thumbnails na esquerda
5. Ter botao "EXPORTAR SLIDE" (html2canvas → PNG)
6. Ter botao "EXPORTAR TODOS" (renderiza todos e mostra pra download)
7. Incluir Google Fonts (Space Grotesk + Space Mono)
8. Incluir html2canvas via CDN
9. Ser 100% standalone (abre em qualquer navegador, sem dependencias locais)

**Elementos visuais disponiveis no template:**
- `.headline-mega/large/medium/small` — hierarquia tipografica
- `.bp-list` + `.bp-marker` — bullets com markers losango
- `.callout` / `.callout.danger` / `.callout.cyan` — caixas de destaque
- `.flow-chain` + `.flow-node` + `.flow-arrow` — fluxos de processo
- `.icon-row` + `.icon-block` — blocos com emoji + label
- `.arrow-down` — seta vertical entre elementos
- `.separator` — linha horizontal gradiente
- `.dim-line` — linha de dimensao com label
- `.slide-footer` + `.footer-handle` + `.footer-dots` — rodape com {{perfil}} e navegacao
- `.slide-hook` — layout centralizado pro slide 1
- `.slide-cta` — layout centralizado pro slide final
- Cores: `.text-amber`, `.text-orange`, `.text-danger`, `.text-cyan`, `.text-dim`

## Passo 5 — Gerar a caption

Gere 2 opcoes de caption:

**Opcao A — Curta + CTA (pro TikTok / alcance)**
- <300 chars
- 1-2 frases de posicao + CTA ManyChat

**Opcao B — Opiniao + estrutura (pro IG / autoridade)**
- 100-240 palavras
- Hook curto → opiniao → contexto → CTA
- Tom: 1a pessoa, provocador ou intimo
- NUNCA descritivo ("neste carrossel eu mostro...")

Regras fixas:
- CTA ManyChat em TODA caption ("comenta [PALAVRA] que te mando")
- Zero hashtags OU 3-4 de nicho max
- Palavras-gatilho: "ninguem fala", "erro", "cuidado", "gratis", numeros
- NUNCA: "top 5", "voce precisa saber", linguagem de guru

Formato de cada opcao:

```
CAPTION [A/B]:
---
[texto completo, pronto pra copiar e colar]
---
Tamanho: [N palavras]
CTA ManyChat: "[frase]"
Palavra-gatilho: [palavra que ativa o bot]
```

ESPERE o {{criador}} escolher.

## Passo 6 — Resumo final

```
CARROSSEL PRONTO

Tema: [titulo]
Pilar: [X]
Slides: [N]
Arquivo: {{materiais}}/carrossel-YYYY-MM-DD-[tema].html

Como usar:
1. Abrir o HTML no navegador
2. Navegar pelos slides (setas ou teclado)
3. Clicar "EXPORTAR TODOS" pra baixar os PNGs
4. Postar no IG com a caption escolhida + musica

Proximos passos:
- /social-copy — se quiser gerar headlines alternativas
- /social-carrossel — pra produzir o proximo
```

## Auto-avaliacao (executar sempre ao final)

Avalie com base nestas perguntas:
1. O slide 1 tem hook de 3-7 palavras (nao descritivo)?
2. Cada slide tem 1 ideia (nao poluido)?
3. A narrativa tem arco claro (problema → desenvolvimento → conclusao)?
4. O HTML renderiza corretamente e exporta PNG?
5. A caption tem opiniao (nao e so descritiva)?
6. O CTA ManyChat e especifico pro tema?
7. A estetica Blueprint esta consistente em todos os slides?
8. O {{criador}} mudou muito da proposta? Se sim, onde o julgamento falhou?

Se identificar melhorias CONCRETAS e EVIDENCIADAS:

```
[AUTO-AVALIACAO]
- [melhoria 1]
- [melhoria 2]
Quer que eu aplique essas melhorias na skill? (s/n)
```

Se nao identificar nada concreto, nao mostre nada.
NAO melhore por melhorar.
