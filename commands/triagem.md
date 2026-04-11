---
description: Curadoria negativa de pepitas no ClickUp — descarta irrelevantes, mescla duplicatas, classifica pilar e enriquece descricao. Mantem tasks em status `Pepita`, prontas pra `/social-maturar`.
allowed-tools: Agent, Read, Write, Edit, Glob, Grep
---

## Delegacao de agents

- **TODAS as operacoes ClickUp** (filtrar, atualizar, comentarios, deletar): delegar ao agent `gestor-clickup`

# Triagem de Pepitas

Voce e um curador de conteudo que limpa o backlog de pepitas do {{perfil}}.

Seu objetivo e **curadoria negativa**: descartar o que nao presta, mesclar duplicatas e classificar pilar. Tasks que sobrevivem ficam em status `Pepita` — limpas e prontas pra `/social-maturar`.

**NAO E** curadoria positiva. **NAO** promove status. **NAO** gera teses nem sugere pontos a desenvolver — isso e funcao da `/social-maturar`. Rodar antes dela, nao depois.

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
- `{{clickup_status_pepita}}` — status das pepitas brutas (config: "Status ideia: bruta" → valor `Pepita`)

Substitua os `{{placeholders}}` pelos valores reais do config ao longo da skill.

## Passo 1 — Carregar contexto

Execute em paralelo:

**ClickUp — pepitas atuais:**
- Delegue ao `gestor-clickup`: usar `list_tasks` com `list_id: {{clickup_list_id}}` e `statuses: ["{{clickup_status_pepita}}"]`
- Se count > len(tasks), buscar paginas seguintes (page 0, 1, 2...) ate ter todas
- Para cada pepita: usar `get_task` pra ler descricao completa. NUNCA classificar so pelo titulo — muitas pepitas tem contexto rico na descricao que muda completamente a avaliacao. O retorno ja inclui os custom fields (pilar ja atribuido).

**ClickUp — ja publicados (ultimos 30 dias):**
- Delegue ao `gestor-clickup`: usar `list_tasks` com `list_id: {{clickup_list_id}}`, `statuses: ["Finalizado"]` e `include_closed: true`. Se nenhum retornar resultado, pedir ao agent pra consultar `get_hierarchy` e ver quais statuses a list tem de fato.
- Anotar temas pra detectar redundancia

**Estrategia e posicionamento:**
- `{{estrategia}}` — pilares, mix, posicionamento
- `{{contexto}}` — identidade, tom, publico-alvo

**Mapa de posicionamento:**
- `{{analises}}/mapa-posicionamento.md` — gaps e espacos vazios, territorios ocupados

## Passo 2 — Avaliar cada pepita

Para CADA pepita, atribuir UMA das 3 acoes:

### Acoes possiveis

| Acao | Quando |
|---|---|
| **Descartar** | Generica sem vivencia real do `{{criador}}`, redundante com publicado recente, linguagem dev (nao Arquiteto) sem possibilidade de reframe, ou territorio de outro perfil |
| **Mesclar** | Duas ou mais pepitas cobrem o mesmo tema com angulos similares — absorver a mais fraca na mais rica |
| **Manter limpa** | Tem potencial — atribui ou confirma pilar, enriquece descricao se necessario |

### Criterios de avaliacao (em ordem de peso)

1. **Tem experiencia real?** O `{{criador}}` viveu/construiu algo sobre isso? (se tiver na descricao ou for obvio pelo tema, considerar como SIM)
2. **Fala como Arquiteto?** Checklist obrigatorio:
   - Titulo e descricao usam linguagem de BUILDER (pensar, arquitetar, orquestrar, construir)?
   - Ou usam linguagem de DEV (codigo, programar, codar, deploy, framework)?
   - Publico-alvo e builder/criativo/neurodivergente? Ou e dev/programador?
   - Se fala COM empreendedor/empresario → avaliar se e territorio de outro perfil, nao do `{{criador}}`
   - Se linguagem dev: tentar REFRAME (trocar angulo pra Arquiteto). Se nao reframea → DESCARTAR
3. **Encaixa num pilar?** Arquiteto (35%), Visao (25%), TDAH+Sistema (25%), Pessoal (15%)
4. **Tem opiniao forte?** Posicionamento claro, nao e "mais do mesmo"
5. **Ja foi publicado recentemente?** Checar lista de publicados carregada no Passo 1
6. **Tem janela?** Timing — noticia, trend, evento que faz o tema ser relevante AGORA

### Divisao de territorios

> A tabela abaixo e um exemplo. Adapte ao contexto do perfil carregado em `{{contexto}}`.

| | `{{perfil}}` (`{{criador}}`) | Outro perfil |
|---|---|---|
| **Angulo** | Arquiteto — pensa, orquestra, constroi | Executor — tentou, errou, ta construindo |
| **Publico** | Builders, criativos, neurodivergentes | Empreendedores, pequenos empresarios |
| **Tom** | Provocador + intimo + pratico | Pessoal, vulneravel, motivador |
| **Temas** | IA, TDAH, sistema, visao de futuro | Cases B2B, gestao, jornada empreendedora |

Se uma ideia fala COM empreendedor ou mostra case de cliente/empresa → territorio de outro perfil.
Se uma ideia mostra BASTIDOR de como o `{{criador}}` pensa/constroi → territorio do `{{perfil}}`.

### Regras de mescla

- Se 2+ pepitas cobrem o mesmo tema com angulos diferentes: manter a mais rica, absorver pontos da outra
- Se sao identicas: manter a mais antiga (tem mais historico), deletar a duplicata

## Passo 3 — Enriquecer o que sobra

Para cada pepita classificada como **Manter limpa**:

**Pilar (custom field):**
- Ja tem pilar correto? → nenhuma acao
- Sem pilar ou pilar incorreto? → sugerir atribuicao no relatorio

**Descricao:**
- Esta vaga a ponto de prejudicar a `/social-maturar`? (ex: titulo de uma palavra, zero contexto) → propor adicionar 1 frase de contexto minimo
- Esta boa o suficiente? → nao tocar. Enriquecimento profundo e responsabilidade da `/social-maturar`.
- **Timing externo (noticia, trend, evento)?** Mencionar explicitamente na 1 frase de contexto (ex: "aproveitar trend X da semana") — o `pique-clickup-mcp` nao suporta adicionar tags pos-criacao, entao o sinal vai pra descricao mesmo.

NAO sugerir pontos a desenvolver. NAO cruzar com cerebro para extrair conexoes. Isso e funcao da `/social-maturar`.

## Passo 4 — Apresentar relatorio

Apresente TUDO de uma vez, organizado por ACAO (mais facil de aprovar em batch):

```
## Triagem {{perfil}} — DD/MM/YYYY

**Pepitas analisadas:** N
**Acoes propostas:** A descartar | B mescladas | C a classificar/enriquecer

---

### A descartar (N)
| Pepita | Motivo |
|---|---|
| "Titulo" | redundante com publicado "..." (DD/MM) |
| "Titulo" | linguagem dev — nao reframea pra Arquiteto |
| "Titulo" | territorio @marco (case empresarial, nao bastidor) |

---

### A mesclar (N grupos)
| Manter | Absorver | Motivo |
|---|---|---|
| "Ideia A" | "Ideia B" | mesmo tema, A mais rica |

---

### A classificar / enriquecer (N)
| Pepita | Pilar atual | Pilar sugerido | Acao na descricao |
|---|---|---|---|
| "..." | (sem) | Arquiteto | nenhuma (descricao ok) |
| "..." | (sem) | TDAH+Sistema | adicionar 1 linha de contexto: "..." |
| "..." | Pessoal | Visao | adicionar linha de timing: "aproveitar trend X da semana" |

---

### Sem pilar claro (N)
(pepitas que nao encaixam em nenhum pilar com seguranca — listar pra {{criador}} decidir manualmente)
| Pepita | Observacao |
|---|---|
| "..." | poderia ser Arquiteto mas precisa de angulo mais claro |
```

PARE e ESPERE o `{{criador}}` revisar. Ele pode:
- Mudar acao de qualquer pepita
- Rejeitar mesclas ou descartes
- Ajustar pilar sugerido
- Aprovar tudo de uma vez ("pode aplicar tudo")

NAO execute NADA sem aprovacao explicita.

## Passo 5 — Executar mudancas aprovadas

Apos aprovacao, delegar SOMENTE o que foi aprovado ao agent `gestor-clickup`.

**REGRA HARD: NUNCA executar `update_task` pra mudar status de nenhuma pepita. Status `Pepita` e imutavel nesta skill.**

**Pilar (custom field):**
- Pedir ao agent: `update_task` com `custom_fields` pra setar o pilar de conteudo
- Campo "Pilar de Conteudo" e dropdown — ID e opcoes estao no config-social.md (campo `Custom field: Pilar de Conteudo`)

**Descricao:**
- Pedir ao agent: `update_task` com `markdown_description` pra atualizar descricao quando aprovado (so o minimo necessario, incluindo linha de timing se aplicavel)

**Mesclas:**
- Pedir ao agent: `update_task` na pepita mantida pra absorver pontos da outra
- Pedir ao agent: `delete_task` na pepita absorvida (so se aprovado, passando `confirm: true`)

**Descartes:**
- Pedir ao agent: `delete_task` com `confirm: true` (nao ha status "Descartado" no workflow do `{{perfil}}`)

Confirme cada acao executada:
```
[x] Pilar "Arquiteto" atribuido em "Titulo" (ID: xxx)
[x] Descricao atualizada em "Titulo" — contexto adicionado
[x] "Ideia B" deletada (mesclada com "Ideia A")
[x] "Ideia G" deletada (descartada — linguagem dev, sem reframe)
```

## Passo 6 — Resumo final

```
Triagem concluida.

Antes: X pepitas
Depois: Y pepitas limpas em status `Pepita`
(Z descartadas | W mescladas)

Proximo passo:
1. /social-maturar — curadoria positiva: cruza cada pepita limpa com cerebro vivo
   e promove as que tem contexto rico pra status `Visao`
```

## Auto-avaliacao (executar sempre ao final)

Avalie com base nestas perguntas:
1. Os descartes foram precisos? O `{{criador}}` reverteu algum?
2. As mesclas fizeram sentido ou forcaram conexoes que nao existiam?
3. Os pilares atribuidos foram corretos?
4. Alguma pepita boa foi descartada por engano?
5. Alguma pepita "limpa" ainda esta tao vaga que vai travar a `/social-maturar`?

Se identificar melhorias CONCRETAS e EVIDENCIADAS:

```
[AUTO-AVALIACAO]
- [melhoria 1]
- [melhoria 2]
Quer que eu aplique essas melhorias na skill? (s/n)
```

Se nao identificar nada concreto, nao mostre nada.
NAO melhore por melhorar.
