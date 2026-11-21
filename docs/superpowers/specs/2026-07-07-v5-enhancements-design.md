# Melhorias na skill formato-curso-v5, retrofit no curso ssh-basico

Data: 2026-07-07

## Contexto

Depois de publicar o curso "SSH e Chaves SSH" (formato-curso-v5) no GitHub Pages e no
portal, revisamos a experiência real do curso e encontramos 6 pontos. Todos rastreiam
até a skill `formato-curso-v5` (`/home/nmaldaner/.claude/skills/formato-curso-v5/`), não
até erros de autoria específicos deste curso — ou seja, qualquer curso já publicado com
esta skill tem os mesmos problemas.

**A skill não é um repositório git** (`~/.claude/skills/` não está sob controle de versão).
Este spec vive no repositório do curso (`ssh-basico`, que é o projeto ativo desta sessão),
mas as mudanças descritas aqui tocam dois lugares:

1. **A skill** (`~/.claude/skills/formato-curso-v5/assets/*` e `references/*.md`) — pra
   valer pra qualquer curso novo gerado a partir de agora.
2. **Este curso** (`ssh-basico/assets/*`, `curso.html`, `landing.html`) — retrofit manual,
   já que os arquivos já foram copiados e publicados antes desta mudança existir.

## Itens levantados e triagem

| # | Problema | Decisão |
|---|---|---|
| 1 | Nome do curso some no topo em telas <560px | **Sem ação** — comparado com outros cursos publicados (`segunda-opiniao`, `cchooks`): o comportamento é idêntico em todos. Não é uma falha deste curso, é o padrão já aceito em toda a linha. |
| 2/3 | Exemplos só narrados em prosa, sem demonstração real de comando/erro | **Feature nova: componente `.termdemo`** (design abaixo) |
| 4 | Sem mecanismo de "refazer exercícios" (só cartões têm revisão) | **Feature nova: refazer exercícios** (design abaixo) |
| 5 | `.recap-autor`/`.next-action` fora do `.layout`, sem `max-width` — ocupam a tela toda | **Fix mecânico de CSS** |
| 6 | `landing-template.html` nunca mostra o nome do curso no header | **Fix mecânico de HTML** |
| — | Opção de trocar a cor de destaque (hoje fixa em `--accent:#7CE0C6`) | **Fora de escopo deste spec** — o usuário pediu pra tratar depois, à parte |

## Feature 1 — Componente `.termdemo` (demo de terminal)

### Objetivo

Substituir a narrativa em prosa ("Fulano digitou X e viu Y") por uma demonstração real,
na coluna da direita (rail), de um comando rodando — incluindo o par "comando certo" +
"erro comum", quando existir um erro comum plausível para aquele comando (mesmo critério
de dosagem que já vale para `.qerr`: não force o quadro se não houver um erro real e
frequente).

### Onde vive

- Só nos steps que têm um comando real associado (tipicamente os mesmos que já têm
  `<div class="pcodewrap">` no corpo do step, ou um comando citado no texto).
- **Substitui** a figura abstrata (`.fig`) da rail nesses steps específicos — não convive
  com ela no mesmo step. Steps sem comando continuam usando `.fig` normalmente.

### Markup

```html
<!-- dentro de .figstage, no lugar do .fig daquele data-fig -->
<div class="termdemo" data-fig="2">
  <button class="termdemo-reveal">ver o que acontece →</button>
  <div class="termdemo-body" hidden>
    <div class="termdemo-cmd on" data-variant="ok">
      <p class="termdemo-k">comando certo</p>
      <pre class="termdemo-in">$ ssh -V</pre>
      <pre class="termdemo-out">OpenSSH_for_Windows_9.5, LibreSSL 3.9.2</pre>
    </div>
    <div class="termdemo-cmd" data-variant="erro">
      <p class="termdemo-k">erro comum</p>
      <pre class="termdemo-in">$ ssh -v</pre>
      <pre class="termdemo-out termdemo-err">(liga o modo detalhado de depuração —
não mostra a versão; -v minúsculo não é o mesmo que -V maiúsculo)</pre>
    </div>
    <button class="termdemo-toggle">ver o erro comum</button>
  </div>
</div>
```

- `.termdemo-body` começa `hidden`; primeiro clique em `.termdemo-reveal` remove o
  `hidden` e mostra a variante `data-variant="ok"` (classe `.on`).
- `.termdemo-toggle` alterna a classe `.on` entre os dois `.termdemo-cmd`, e troca o
  próprio texto do botão ("ver o erro comum" ↔ "ver o certo").
- Quando não há erro comum plausível pra aquele comando, o step usa só um
  `.termdemo-cmd.on` (sem o segundo bloco nem o botão de toggle) — variante "single".

### CSS (novidade em `aula.css`)

- `.termdemo` ocupa o mesmo espaço/posição que `.figstage .fig` ocupa hoje.
- `.termdemo-reveal` reaproveita o estilo de `.readbtn` (mesmo botão, mesmo peso visual).
- `.termdemo-in`/`.termdemo-out` no estilo mono já usado em `.pcode`, `.termdemo-err` com
  leve destaque em âmbar (`--warm`) no texto — nunca vermelho (mantém a regra DV-4: âmbar
  nunca é "resposta errada" no quiz, mas aqui é claramente rotulado como "erro comum" em
  texto, então não conflita com essa regra).
- Entrada: reaproveita a transição que `.fig` já tem (`opacity`/`transform`), aplicada ao
  `.termdemo` inteiro — nenhum motion novo, só reusar o token existente.

### JS (novidade em `curso.js`)

- Um listener em `.termdemo-reveal`: remove `hidden` de `.termdemo-body`, esconde o botão
  de reveal.
- Um listener em `.termdemo-toggle`: alterna `.on` entre os dois `.termdemo-cmd` e o texto
  do botão.
- Nenhuma persistência em `localStorage` — é um estado de UI efêmero (não precisa lembrar
  se a pessoa já revelou, ao recarregar volta fechado).

### Sem conflito com os tetos existentes

`.termdemo` **não** é um dos "9 componentes + 3 quadros" do delta pedagógico da v5 (nem
está sujeito à regra de contenção AD-7, que é especificamente sobre esses). É uma
substituição de conteúdo dentro do slot de figura por step que **já existe** desde o v4
(cada step já tem exatamente 1 `data-fig`) — `.termdemo` é só uma segunda opção de
conteúdo pra esse mesmo slot (SVG diagrama ou termdemo), escolhida pelo autor por step.
Também não usa nenhum ícone novo (o botão de reveal reaproveita o estilo do `.readbtn`, o
toggle só troca texto) — não conflita com o teto de ≤10 ícones (`V5-DESIGN.md` §3).

### Documentação da skill a atualizar

- `SKILL.md`: documentar `.termdemo` como uma segunda opção de conteúdo pro slot de
  figura do step (ao lado do `.fig`/diagrama-de-mecanismo já existente), com a regra de
  quando usar (step tem comando real + existe erro comum plausível).
- `V5-DESIGN.md` §4 (registros visuais): `.termdemo` é um terceiro registro (nem
  metáfora-do-mundo-real, nem diagrama-de-mecanismo — é literal, mostra a tela real) — vale
  documentar essa distinção explicitamente.
- `CHECKLIST-V5.md`: novo item de verificação — steps com comando real têm `.termdemo`;
  steps sem comando não têm.

## Feature 2 — Refazer exercícios

### Por aula

- Botão "refazer exercício desta aula" na caixa de fim de aula que o motor já injeta
  (mesmo lugar onde já existe "revisar os cartões desta aula", ver `curso.js` linha ~195).
- Ação: zera `state.aulas[k].tasks` (mesmo objeto que os checkboxes de `.psteps` já leem),
  chama de novo a função `refresh()` que o handler de `.practice` já define, e rola a tela
  até a `.practice` daquela aula.

### Trilha inteira

- Botão novo na barra do topo, ao lado de "revisar" e "jornada" (ex.: rótulo "exercícios").
- Abre um painel novo (mesmo padrão visual dos painéis `.panel` que já existem: `revisar`,
  `jornada`) listando as 9 práticas do curso, uma linha por aula:
  - Nome da aula + link "ir até ela" (navega pra `#aula-N` e fecha o painel).
  - Botão "resetar" individual por exercício (mesma ação de reset da versão por aula,
    aplicada a cada `k` da lista).
- Sem "resetar tudo de uma vez" — só individual, por decisão do usuário.

### Estado

Nenhuma estrutura de dados nova — reaproveita `state.aulas[k].tasks` que já existe. O
"reset" é só `state.aulas[k].tasks = {}` seguido de `save()` e um refresh visual.

### Documentação da skill a atualizar

- `SKILL.md`: documentar o novo botão/painel como parte do contrato herdado do motor
  (igual `revisar`/`jornada` já são documentados).

## Fixes mecânicos

### `.recap-autor`/`.next-action` desalinhados

Hoje ambos são filhos diretos de `<section class="view">`, fora da `<div class="layout">`
(que tem `max-width:1160px` + padding). Fix: envolver os dois (mais o `<script
id="cards-N">`, que não é visual mas fica no mesmo bloco) num wrapper com o mesmo
`max-width`/padding — seja reaproveitando `.layout` mesmo (só a coluna, sem a rail), seja
uma classe nova `.fecho-wrap` com as mesmas regras de largura. Aplicar tanto no
`curso-template.html` da skill quanto no `curso.html` deste curso.

### Nome do curso ausente no header do `landing.html`

`landing-template.html` (skill) e `landing.html` (este curso) só mostram
`INEMA.CLUB` + `PRO` no header. Fix: adicionar um terceiro item, o nome do curso, no
mesmo padrão visual do `.course` que já existe no `curso.html` (link mudo, sem href — ou
com href pra própria landing, já que é a home).

## Fora de escopo

- Opção de cor de destaque customizável (`--accent`) — combinado que fica pra depois.
- Item 1 (nome do curso sumindo <560px) — sem ação, comportamento já é consistente com o
  resto da linha de cursos.

## Plano de rollout

1. Implementar tudo primeiro na **skill** (`~/.claude/skills/formato-curso-v5/assets/*` +
   `references/*.md` + `SKILL.md`), já que é a fonte de verdade pra cursos futuros.
2. Retrofit no **ssh-basico**: copiar os `assets/aula.css`/`assets/curso.js` atualizados,
   adicionar `.termdemo` nos steps que já têm `.pcodewrap` hoje — conferido diretamente no
   `curso.html`: aula 2 (step 1), aula 3 (steps 1 e 3), aula 6 (steps 1 e 3), aula 7 (steps
   2, 3 e 4), aula 8 (step 2), aula 9 (step 4). Aulas 1, 4 e 5 não têm nenhum step com
   comando real hoje, então não recebem `.termdemo` neste retrofit — continuam com `.fig`
   normalmente. Envolver `.recap-autor`/`.next-action` no wrapper corrigido em todas as 9
   aulas, e atualizar `landing.html` com o nome do curso no header.
3. Re-rodar o checklist-portão (os greps já usados antes: promise/tempo/psafe/recap/
   next-action/cards, densidade visual, anti-duplicação) pra garantir que o retrofit não
   quebrou nenhum dos 9/9 que já passavam.
4. Testar visualmente (abrir no navegador) pelo menos 1 aula com `.termdemo` e o painel
   novo de exercícios antes de publicar.
5. Commit + push do curso; a skill não tem git, então não há o que commitar lá — só
   deixar os arquivos atualizados em `~/.claude/skills/formato-curso-v5/`.
