# v5 Enhancements + ssh-basico Retrofit Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a `.termdemo` (terminal-demo) component and a "refazer exercícios" review mechanism to the `formato-curso-v5` skill, fix two CSS/HTML bugs (recap alignment, missing course name on landing), and retrofit all of it into the already-published `ssh-basico` course.

**Architecture:** Plain HTML/CSS/vanilla-JS, no build step, no framework — matches the existing skill's engine exactly (`curso.js` is one IIFE, `aula.css` is one stylesheet, both hand-written and shared verbatim across all v5 courses). `.termdemo` reuses the existing `.fig` positioning/animation machinery by sharing its CSS class. The exercise-review mechanism reuses the existing `state.aulas[k].tasks` data already used by `.practice` checkboxes — no new state shape.

**Tech Stack:** HTML5, CSS3 (custom properties), vanilla ES5-style JS (`curso.js` avoids arrow functions/const to stay compatible with the rest of the file), Python 3 (only for the one scripted bulk-edit task), git.

## Global Constraints

- No test framework exists in this codebase (static HTML/CSS/JS, no npm/build step) — "tests" in this plan are grep-based structural checks (mirroring the checklist-portão greps already used to validate `ssh-basico`) plus one manual browser check at the end. There is no `pytest`/`jest` to run.
- Two locations must both be edited for every skill-level change: the skill itself (`~/.claude/skills/formato-curso-v5/` — a **symlink** to `~/projetos/formato-curso-inema/formato-curso-v5/`, which IS a git repo; work happens on branch `v5-enhancements`, off `main`) and the course (`~/projetos/ssh-basico/`, a separate git repo, worked on directly on `main`). Editing through the symlink path writes through to the real file — either path works, but commits happen in `~/projetos/formato-curso-inema/`.
- Every task commits its own change individually (standard implementer step 4) — this keeps each task reviewer's diff scoped to exactly that task, per `subagent-driven-development`'s design. Task 9 additionally pushes `formato-curso-inema`'s `v5-enhancements` branch (closing out Part A); Task 20 additionally pushes `ssh-basico`'s `main` (closing out Part B) — intermediate tasks commit locally without pushing.
- `curso.js` is a single IIFE using `var`, `function(){}` and jQuery-less DOM APIs throughout — new code must match this style (no `const`/`let`/arrow functions/template literals), or it will look inconsistent next to 800+ existing lines in the same style.
- Never touch the router, `norm()`/base state, trilho (`activateFor`/IntersectionObserver), grifo/highlight, or SM-2 quiz logic — per `SKILL.md`'s "Proibido tocar" list. All new code is additive.
- `.termdemo` must carry the `fig` class too (`class="fig termdemo"`) in the rail so it's picked up for free by the existing trilho fade-in/highlight logic — do not write new JS for its entrance animation.
- Every `.termdemo` step needs BOTH a rail copy (inside `.figstage`, class `fig termdemo`) AND a mobile copy (inside `.mobfig`, class `termdemo` only, no `fig` class) — the existing SVG-cloning JS only clones `<svg>` elements, so it silently no-ops for `.termdemo` (no `<svg>` to find) and the author-written mobile copy stands as-is.

---

## Part A — Skill (`~/.claude/skills/formato-curso-v5/`)

### Task 1: `.termdemo` CSS

**Files:**
- Modify: `~/.claude/skills/formato-curso-v5/assets/aula.css` (append at end of file)

**Interfaces:**
- Produces: CSS classes `.termdemo`, `.fig.termdemo`, `.termdemo-reveal`, `.termdemo-body`, `.termdemo-cmd`, `.termdemo-cmd.on`, `.termdemo-k`, `.termdemo-in`, `.termdemo-out`, `.termdemo-out.termdemo-err`, `.termdemo-toggle` — consumed by Task 4 (JS) and Task 7 (template markup).

- [ ] **Step 1: Append the CSS block**

Append this exact block to the end of `~/.claude/skills/formato-curso-v5/assets/aula.css`:

```css

/* ---- termdemo: demo real de comando, 2ª opção pro slot de figura do step (v5 delta) ---- */
.termdemo{width:100%;display:flex;flex-direction:column;align-items:center;justify-content:center;gap:14px}
.fig.termdemo{position:absolute;inset:0;padding:24px}
.termdemo-reveal{font-family:var(--mono);font-size:12px;letter-spacing:.04em;color:var(--muted);background:var(--panel2);border:1px solid var(--line2);border-radius:99px;padding:10px 18px;cursor:pointer;transition:border-color .2s,color .2s}
.termdemo-reveal:hover{border-color:var(--accent);color:var(--ink)}
.termdemo-body{width:100%}
.termdemo-body[hidden]{display:none}
.termdemo-cmd{display:none;width:100%}
.termdemo-cmd.on{display:block}
.termdemo-k{font-family:var(--mono);font-size:11px;letter-spacing:.14em;text-transform:uppercase;color:var(--accent);margin:0 0 8px}
.termdemo-cmd[data-variant="erro"] .termdemo-k{color:var(--warm)}
.termdemo-in{font-family:var(--mono);font-size:13.5px;color:var(--ink);background:var(--panel2);border:1px solid var(--line2);border-radius:10px 10px 0 0;padding:10px 14px;margin:0;white-space:pre-wrap}
.termdemo-out{font-family:var(--mono);font-size:12.5px;color:var(--ink-soft);background:var(--bg2);border:1px solid var(--line2);border-top:none;border-radius:0 0 10px 10px;padding:10px 14px;margin:0 0 14px;white-space:pre-wrap}
.termdemo-out.termdemo-err{color:var(--warm)}
.termdemo-toggle{font-family:var(--mono);font-size:11px;letter-spacing:.04em;color:var(--muted);background:none;border:1px solid var(--line2);border-radius:99px;padding:8px 14px;cursor:pointer;transition:border-color .2s,color .2s}
.termdemo-toggle:hover{border-color:var(--accent);color:var(--ink)}
```

- [ ] **Step 2: Verify the block landed correctly**

Run: `grep -c 'termdemo' ~/.claude/skills/formato-curso-v5/assets/aula.css`
Expected: `17` (one for each class reference above, `.fig.termdemo` counts as 1 for `termdemo`).

- [ ] **Step 3: Commit**

```bash
cd ~/projetos/formato-curso-inema
git add formato-curso-v5/assets/aula.css
git commit -m "feat(v5): add .termdemo CSS (terminal-demo component)"
```

---

### Task 2: `.fecho-wrap` CSS (fixes the recap alignment bug)

**Files:**
- Modify: `~/.claude/skills/formato-curso-v5/assets/aula.css` (append)

**Interfaces:**
- Produces: `.fecho-wrap` class — consumed by Task 7 (template) and Task 10 (course retrofit).

- [ ] **Step 1: Append the CSS block**

```css

/* ---- fecho-wrap: alinha recap-autor/next-action com a largura do .layout (fix desalinhamento) ---- */
.fecho-wrap{max-width:1160px;margin:0 auto;padding:0 clamp(24px,5vw,60px)}
.fecho-wrap .recap-autor,.fecho-wrap .next-action{max-width:var(--read)}
```

- [ ] **Step 2: Verify**

Run: `grep -c 'fecho-wrap' ~/.claude/skills/formato-curso-v5/assets/aula.css`
Expected: `3`

---

### Task 3: Exercícios panel CSS

**Files:**
- Modify: `~/.claude/skills/formato-curso-v5/assets/aula.css` (append)

**Interfaces:**
- Produces: `.exlist`, `.exrow`, `.exinfo`, `.ext`, `.exstat`, `.exstat.done`, `.exactions`, `.exgo`, `.exreset` — consumed by Task 6 (JS `renderExercicios`).

- [ ] **Step 1: Append the CSS block**

```css

/* ---- painel de exercícios (refazer exercícios da trilha inteira) ---- */
.exlist{display:flex;flex-direction:column;gap:10px}
.exrow{display:flex;align-items:center;justify-content:space-between;gap:14px;padding:14px 16px;border:1px solid var(--line);border-radius:14px;background:var(--panel)}
.exinfo{display:flex;flex-direction:column;gap:4px}
.ext{margin:0;font-size:14.5px;color:var(--ink)}
.exstat{margin:0;font-family:var(--mono);font-size:11px;letter-spacing:.06em;text-transform:uppercase;color:var(--muted)}
.exstat.done{color:var(--accent)}
.exactions{display:flex;gap:8px;flex-shrink:0}
.exgo,.exreset{font-family:var(--mono);font-size:11px;color:var(--ink-soft);background:var(--panel2);border:1px solid var(--line2);border-radius:99px;padding:8px 12px;cursor:pointer;transition:border-color .2s,color .2s}
.exgo:hover,.exreset:hover{border-color:var(--accent);color:var(--ink)}
```

- [ ] **Step 2: Verify**

Run: `grep -c 'exrow\|exlist\|exinfo\|exactions\|exgo\|exreset\|exstat' ~/.claude/skills/formato-curso-v5/assets/aula.css`
Expected: a number ≥ `9` (each selector line may match more than once via the combined selectors above; just confirm it's non-zero and the file has no syntax break — see Step 3).

- [ ] **Step 3: Sanity-check the CSS file has no unbalanced braces**

Run:
```bash
python3 -c "
s = open('$HOME/.claude/skills/formato-curso-v5/assets/aula.css').read()
print('open:', s.count('{'), 'close:', s.count('}'))
"
```
Expected: both numbers equal (any mismatch means a typo was introduced in Tasks 1–3).

---

### Task 4: `.termdemo` JS behavior (reveal + toggle)

**Files:**
- Modify: `~/.claude/skills/formato-curso-v5/assets/curso.js`

**Interfaces:**
- Consumes: markup from Task 1 (`.termdemo-reveal`, `.termdemo-body`, `.termdemo-cmd`, `.termdemo-toggle`).
- Produces: nothing consumed by other tasks (self-contained UI behavior).

- [ ] **Step 1: Locate the insertion point**

Run: `grep -n "teste-se ----" ~/.claude/skills/formato-curso-v5/assets/curso.js`
Expected output includes a line like `198:  // ---- teste-se ----` — insert the new block immediately **before** that line (i.e., right after the `.practice` forEach block that ends around line 179, and before the `// ---- teste-se ----` comment).

- [ ] **Step 2: Insert the termdemo JS block**

Using the Edit tool, find this exact existing text in `curso.js`:

```
  // copiar em qualquer bloco de código
  document.querySelectorAll('.view[data-aula] .col pre').forEach(function(pre){ if(pre.closest('.pcodewrap')||pre.closest('.codewrap')) return; var wrap=el('div','codewrap'); pre.parentNode.insertBefore(wrap,pre); wrap.appendChild(pre); var b=el('button','pcopy'); b.type='button'; b.textContent='copiar'; wrap.insertBefore(b,pre); b.addEventListener('click',function(){ copyText(pre.textContent,b); }); });
```

Replace it with this (original line kept, new block appended after it):

```
  // copiar em qualquer bloco de código
  document.querySelectorAll('.view[data-aula] .col pre').forEach(function(pre){ if(pre.closest('.pcodewrap')||pre.closest('.codewrap')) return; var wrap=el('div','codewrap'); pre.parentNode.insertBefore(wrap,pre); wrap.appendChild(pre); var b=el('button','pcopy'); b.type='button'; b.textContent='copiar'; wrap.insertBefore(b,pre); b.addEventListener('click',function(){ copyText(pre.textContent,b); }); });

  // ---- termdemo (demo real de comando: clique-pra-revelar + toggle certo/erro) ----
  document.querySelectorAll('.termdemo-reveal').forEach(function(b){
    b.addEventListener('click',function(){
      var body=b.nextElementSibling; if(!body) return;
      body.hidden=false; b.style.display='none';
    });
  });
  document.querySelectorAll('.termdemo-toggle').forEach(function(b){
    b.addEventListener('click',function(){
      var body=b.closest('.termdemo-body'); if(!body) return;
      var cmds=[].slice.call(body.querySelectorAll('.termdemo-cmd'));
      var okCmd=cmds.filter(function(c){ return c.getAttribute('data-variant')==='ok'; })[0];
      var errCmd=cmds.filter(function(c){ return c.getAttribute('data-variant')==='erro'; })[0];
      if(!okCmd||!errCmd) return;
      var showingOk=okCmd.classList.contains('on');
      cmds.forEach(function(c){ c.classList.remove('on'); });
      if(showingOk){ errCmd.classList.add('on'); b.textContent='ver o certo'; }
      else { okCmd.classList.add('on'); b.textContent='ver o erro comum'; }
    });
  });
```

- [ ] **Step 3: Verify with Node (syntax check only, no DOM available)**

Run: `node --check ~/.claude/skills/formato-curso-v5/assets/curso.js`
Expected: no output, exit code 0 (this only validates JS syntax parses; it does not run the DOM-dependent code).

---

### Task 5: `practiceRefreshers` + `resetPractice()` + per-aula "refazer exercício" button

**Files:**
- Modify: `~/.claude/skills/formato-curso-v5/assets/curso.js`

**Interfaces:**
- Produces: `practiceRefreshers` (object, keyed by aula id `k`, values are the existing per-practice `refresh` closures), `resetPractice(k, scroll)` function — consumed by Task 6 (exercícios panel) and by the per-aula button added in this same task.

- [ ] **Step 1: Expose the refresh closures**

Using Edit, find this exact block in `curso.js`:

```
  document.querySelectorAll('.practice').forEach(function(pr){ var k=aulaOf(pr); if(!k) return; var tasks=state.aulas[k].tasks, boxes=[].slice.call(pr.querySelectorAll('input[data-ptask]')), cnt=pr.querySelector('.pcount');
    function refresh(){ var done=0; boxes.forEach(function(b){ var id=b.getAttribute('data-ptask'); b.checked=!!tasks[id]; if(tasks[id]) done++; }); if(cnt) cnt.textContent=done+'/'+boxes.length+' feito'; pr.classList.toggle('done', boxes.length>0&&done===boxes.length); }
    boxes.forEach(function(b){ b.addEventListener('change',function(){ tasks[b.getAttribute('data-ptask')]=b.checked; save(); refresh(); }); });
    var cp=pr.querySelector('.pcopy'), code=pr.querySelector('.pcode'); if(cp&&code) cp.addEventListener('click',function(){ copyText(code.textContent,cp); });
    refresh();
  });
```

Replace with:

```
  var practiceRefreshers={};
  document.querySelectorAll('.practice').forEach(function(pr){ var k=aulaOf(pr); if(!k) return; var tasks=state.aulas[k].tasks, boxes=[].slice.call(pr.querySelectorAll('input[data-ptask]')), cnt=pr.querySelector('.pcount');
    function refresh(){ var done=0; boxes.forEach(function(b){ var id=b.getAttribute('data-ptask'); b.checked=!!tasks[id]; if(tasks[id]) done++; }); if(cnt) cnt.textContent=done+'/'+boxes.length+' feito'; pr.classList.toggle('done', boxes.length>0&&done===boxes.length); }
    boxes.forEach(function(b){ b.addEventListener('change',function(){ tasks[b.getAttribute('data-ptask')]=b.checked; save(); refresh(); }); });
    var cp=pr.querySelector('.pcopy'), code=pr.querySelector('.pcode'); if(cp&&code) cp.addEventListener('click',function(){ copyText(code.textContent,cp); });
    practiceRefreshers[k]=refresh;
    refresh();
  });
  function resetPractice(k,scroll){
    var tasks=state.aulas[k].tasks; if(!tasks) return;
    Object.keys(tasks).forEach(function(id){ tasks[id]=false; });
    save();
    if(practiceRefreshers[k]) practiceRefreshers[k]();
    if(scroll){ var v=aulaEls[k], pr=v&&v.querySelector('.practice'); if(pr) pr.scrollIntoView({behavior:'smooth',block:'start'}); }
  }
```

- [ ] **Step 2: Add the per-aula "refazer exercício" button**

Using Edit, find this exact block (the per-aula recap box injection):

```
  // ---- recap ativo no fim da aula (fecha com recordação) ----
  document.querySelectorAll('.view[data-aula] .col').forEach(function(col){ var k=aulaOf(col); if(!k||col.querySelector('.recap')) return;
    var box=el('div','recap'); box.innerHTML='<p class="rk">Feche a aula</p><p class="rtext">Revise agora, com recorda&ccedil;&atilde;o ativa &mdash; depois estes cart&otilde;es voltam sozinhos na hora certa. A revis&atilde;o da trilha (bot&atilde;o <b>revisar</b> no topo) mistura as perguntas de <b>todas</b> as aulas.</p>';
    var b=el('button','big'); b.textContent='revisar os cartões desta aula'; b.addEventListener('click',function(){ reviewAula(k); }); box.appendChild(b); col.appendChild(box);
  });
```

Replace with:

```
  // ---- recap ativo no fim da aula (fecha com recordação) ----
  document.querySelectorAll('.view[data-aula] .col').forEach(function(col){ var k=aulaOf(col); if(!k||col.querySelector('.recap')) return;
    var box=el('div','recap'); box.innerHTML='<p class="rk">Feche a aula</p><p class="rtext">Revise agora, com recorda&ccedil;&atilde;o ativa &mdash; depois estes cart&otilde;es voltam sozinhos na hora certa. A revis&atilde;o da trilha (bot&atilde;o <b>revisar</b> no topo) mistura as perguntas de <b>todas</b> as aulas.</p>';
    var b=el('button','big'); b.textContent='revisar os cartões desta aula'; b.addEventListener('click',function(){ reviewAula(k); }); box.appendChild(b);
    if(aulaEls[k]&&aulaEls[k].querySelector('.practice')){
      var b2=el('button','big'); b2.style.marginLeft='10px'; b2.textContent='refazer exercício desta aula'; b2.addEventListener('click',function(){ resetPractice(k,true); }); box.appendChild(b2);
    }
    col.appendChild(box);
  });
```

- [ ] **Step 3: Verify with Node syntax check**

Run: `node --check ~/.claude/skills/formato-curso-v5/assets/curso.js`
Expected: no output, exit code 0.

- [ ] **Step 4: Verify the exact strings are present**

Run: `grep -c "practiceRefreshers\|resetPractice\|refazer exercício" ~/.claude/skills/formato-curso-v5/assets/curso.js`
Expected: `6` (2 `practiceRefreshers` + 3 `resetPractice` + 1 `refazer exercício`).

---

### Task 6: Exercícios panel (whole-trilha "refazer exercícios")

**Files:**
- Modify: `~/.claude/skills/formato-curso-v5/assets/curso.js`

**Interfaces:**
- Consumes: `practiceRefreshers`/`resetPractice` (Task 5), `praticaDone(k)` (already exists at line ~105), `KEYS`/`aulaEls`/`el`/`$`/`openPanel`/`closePanels` (already exist).
- Produces: `renderExercicios()` function, `#exercicios` panel, `#exbtn` listener.

- [ ] **Step 1: Inject the panel markup**

Using Edit, find this exact line in `curso.js` (part of the big `insertAdjacentHTML('beforeend', ...)` call):

```
    '<aside class="panel" id="revisar"><button class="close" data-close>&#10005;</button><h3>Revisar</h3><div class="rev" id="revbody"></div></aside>'+
```

Replace with:

```
    '<aside class="panel" id="revisar"><button class="close" data-close>&#10005;</button><h3>Revisar</h3><div class="rev" id="revbody"></div></aside>'+
    '<aside class="panel" id="exercicios"><button class="close" data-close>&#10005;</button><h3>Exerc&iacute;cios</h3><div class="exlist" id="exlist"></div></aside>'+
```

- [ ] **Step 2: Add `renderExercicios()` right after `renderJornada()`**

Using Edit, find this exact closing of `renderJornada()`:

```
    var mk=$('jmarks'), all=[]; KEYS.forEach(function(k){ state.aulas[k].marks.forEach(function(m){ all.push(m); }); });
    if(!all.length) mk.innerHTML='<p style="color:var(--muted);font-size:14px">Nenhum grifo ainda.</p>'; else { mk.innerHTML=''; all.slice().reverse().forEach(function(m){ var d=el('div','mk'); d.textContent='“'+m.text+'”'; mk.appendChild(d); }); }
  }
```

Replace with:

```
    var mk=$('jmarks'), all=[]; KEYS.forEach(function(k){ state.aulas[k].marks.forEach(function(m){ all.push(m); }); });
    if(!all.length) mk.innerHTML='<p style="color:var(--muted);font-size:14px">Nenhum grifo ainda.</p>'; else { mk.innerHTML=''; all.slice().reverse().forEach(function(m){ var d=el('div','mk'); d.textContent='“'+m.text+'”'; mk.appendChild(d); }); }
  }

  // ---- exercícios (refazer, trilha inteira) ----
  function renderExercicios(){
    var box=$('exlist'); if(!box) return; box.innerHTML='';
    KEYS.forEach(function(k){
      var v=aulaEls[k], pr=v&&v.querySelector('.practice'); if(!pr) return;
      var h=pr.querySelector('.ph'), title=h?h.textContent.trim():('Aula '+k);
      var row=el('div','exrow');
      var info=el('div','exinfo');
      var t=el('p','ext'); t.textContent='Aula '+k+' — '+title; info.appendChild(t);
      var done=praticaDone(k);
      var st=el('p','exstat'+(done?' done':'')); st.textContent=done?'feito':'pendente'; info.appendChild(st);
      row.appendChild(info);
      var actions=el('div','exactions');
      var go=el('button','exgo'); go.type='button'; go.textContent='ir até ela'; go.addEventListener('click',function(){ closePanels(); location.hash='aula-'+k; });
      var rs=el('button','exreset'); rs.type='button'; rs.textContent='resetar'; rs.addEventListener('click',function(){ resetPractice(k,false); renderExercicios(); });
      actions.appendChild(go); actions.appendChild(rs);
      row.appendChild(actions);
      box.appendChild(row);
    });
    if(!box.children.length) box.innerHTML='<p style="color:var(--muted);font-size:14px">Nenhum exercício nesta trilha.</p>';
  }
  if($('exbtn')) $('exbtn').addEventListener('click',function(){ renderExercicios(); openPanel('exercicios'); });
```

- [ ] **Step 3: Verify with Node syntax check**

Run: `node --check ~/.claude/skills/formato-curso-v5/assets/curso.js`
Expected: no output, exit code 0.

- [ ] **Step 4: Verify strings present**

Run: `grep -c "renderExercicios\|id=\"exercicios\"\|exbtn" ~/.claude/skills/formato-curso-v5/assets/curso.js`
Expected: `5` (3 `renderExercicios` + 1 `id="exercicios"` + 1 `exbtn`... count may vary slightly by exact occurrences; confirm it is **not 0** and re-read the diff if the count looks wrong).

---

### Task 7: Update `curso-template.html` (bar button, fecho-wrap, termdemo example)

**Files:**
- Modify: `~/.claude/skills/formato-curso-v5/assets/curso-template.html`

**Interfaces:**
- Consumes: CSS/JS from Tasks 1–6.

- [ ] **Step 1: Add the "exercícios" bar button**

Using Edit, find:

```
  <button id="revbtn">revisar <span id="revn">0</span></button>
  <button id="jorbtn">jornada</button>
```

Replace with:

```
  <button id="revbtn">revisar <span id="revn">0</span></button>
  <button id="exbtn">exercícios</button>
  <button id="jorbtn">jornada</button>
```

- [ ] **Step 2: Wrap the example recap-autor/next-action in `.fecho-wrap`**

Using Edit, find:

```
<!-- FECHO — ordem fixa: recap-autor → next-action → cards-N → (motor injeta o resto depois) -->
<div class="recap-autor">
```

Replace with:

```
<!-- FECHO — ordem fixa: recap-autor → next-action → cards-N → (motor injeta o resto depois) -->
<div class="fecho-wrap">
<div class="recap-autor">
```

Then find:

```
<div class="next-action">
  <p class="nak">Seu próximo passo</p>
  <p class="na-win">Você acabou de conseguir [microvitória nomeada].</p>
  <p class="na-action">Nos próximos 15 minutos, no seu trabalho de verdade: [microação concreta].</p>
  <p class="na-hook">Na próxima aula, você resolve [a próxima dor real que trava quem já passou por esta].</p>
</div>
```

Replace with:

```
<div class="next-action">
  <p class="nak">Seu próximo passo</p>
  <p class="na-win">Você acabou de conseguir [microvitória nomeada].</p>
  <p class="na-action">Nos próximos 15 minutos, no seu trabalho de verdade: [microação concreta].</p>
  <p class="na-hook">Na próxima aula, você resolve [a próxima dor real que trava quem já passou por esta].</p>
</div>
</div>
```

- [ ] **Step 3: Replace the example step-2 rail figure with a `.termdemo` example**

Using Edit, find this exact rail figure (step 2's diagram, inside `.figstage`):

```
      <div class="fig" data-fig="2" data-cap="<b>Fig.</b> legenda que ensina da 2ª ideia"><svg viewBox="0 0 220 200" fill="none" aria-hidden="true"><!-- diagrama conceitual --></svg></div>
```

Replace with:

```
      <!-- .termdemo — 2ª opção pro slot de figura: use quando o step tiver um comando real. Sempre com par certo+erro quando fizer sentido (senão, 1 só .termdemo-cmd sem o botão de toggle). -->
      <div class="fig termdemo" data-fig="2" data-cap="<b>Fig.</b> rode o comando e veja a diferença entre o certo e o erro comum">
        <button class="termdemo-reveal" type="button">ver o que acontece →</button>
        <div class="termdemo-body" hidden>
          <div class="termdemo-cmd on" data-variant="ok">
            <p class="termdemo-k">comando certo</p>
            <pre class="termdemo-in">$ comando --certo</pre>
            <pre class="termdemo-out">saída esperada</pre>
          </div>
          <div class="termdemo-cmd" data-variant="erro">
            <p class="termdemo-k">erro comum</p>
            <pre class="termdemo-in">$ comando --errado</pre>
            <pre class="termdemo-out termdemo-err">a mensagem de erro real, e por que ela aparece</pre>
          </div>
          <button class="termdemo-toggle" type="button">ver o erro comum</button>
        </div>
      </div>
```

Also, in the matching step-2 body (in `.col`), replace the empty mobile placeholder:

```
      <div class="mobfig" data-mobfig="2"></div>
```

Replace with:

```
      <!-- cópia mobile do .termdemo acima — mesmo markup, sem a classe "fig" (não é posicionado em rail aqui) -->
      <div class="mobfig">
        <div class="termdemo">
          <button class="termdemo-reveal" type="button">ver o que acontece →</button>
          <div class="termdemo-body" hidden>
            <div class="termdemo-cmd on" data-variant="ok">
              <p class="termdemo-k">comando certo</p>
              <pre class="termdemo-in">$ comando --certo</pre>
              <pre class="termdemo-out">saída esperada</pre>
            </div>
            <div class="termdemo-cmd" data-variant="erro">
              <p class="termdemo-k">erro comum</p>
              <pre class="termdemo-in">$ comando --errado</pre>
              <pre class="termdemo-out termdemo-err">a mensagem de erro real, e por que ela aparece</pre>
            </div>
            <button class="termdemo-toggle" type="button">ver o erro comum</button>
          </div>
        </div>
      </div>
```

- [ ] **Step 4: Verify**

Run: `grep -c 'exbtn\|fecho-wrap\|termdemo' ~/.claude/skills/formato-curso-v5/assets/curso-template.html`
Expected: a number ≥ `15` (non-zero is the real bar; exact count depends on the template's other content — if it's `0`, the edits did not land).

---

### Task 8: Landing header — add course name

**Files:**
- Modify: `~/.claude/skills/formato-curso-v5/assets/landing-template.html`

**Interfaces:**
- Produces: `header.top .course` CSS class + markup, to be mirrored in the course's own `landing.html` (Task 12).

- [ ] **Step 1: Add the CSS rule**

Using Edit, find:

```
  header.top .pro{background:var(--accent);color:var(--accent-ink);border-radius:4px;padding:2px 6px;font-size:9.5px;font-weight:700;letter-spacing:.1em;text-decoration:none;margin-left:8px}
```

Replace with:

```
  header.top .pro{background:var(--accent);color:var(--accent-ink);border-radius:4px;padding:2px 6px;font-size:9.5px;font-weight:700;letter-spacing:.1em;text-decoration:none;margin-left:8px}
  header.top .course{color:var(--muted);font-size:12px;letter-spacing:.04em;border-left:1px solid var(--line2);padding-left:10px;margin-left:2px;font-family:var(--mono)}
```

- [ ] **Step 2: Add the markup**

Using Edit, find:

```
<header class="top"><div class="wrap">
  <a class="mark" href="https://inema.club" target="_blank" rel="noopener">INEMA.CLUB</a>
  <a class="pro" href="https://inema.pro" target="_blank" rel="noopener">PRO</a>
</div></header>
```

Replace with:

```
<header class="top"><div class="wrap">
  <a class="mark" href="https://inema.club" target="_blank" rel="noopener">INEMA.CLUB</a>
  <a class="pro" href="https://inema.pro" target="_blank" rel="noopener">PRO</a>
  <span class="course">Nome do curso</span>
</div></header>
```

(`Nome do curso` is a literal placeholder here — this file is the skill's authoring template, meant to be filled in per course. This is not a violation of the "no placeholders" plan rule, since this file's entire purpose is placeholder text for future courses — see every other `PLACEHOLDER` string already in it.)

- [ ] **Step 3: Verify**

Run: `grep -c 'class="course"' ~/.claude/skills/formato-curso-v5/assets/landing-template.html`
Expected: `1`

---

### Task 9: Update skill documentation

**Files:**
- Modify: `~/.claude/skills/formato-curso-v5/SKILL.md`
- Modify: `~/.claude/skills/formato-curso-v5/references/V5-DESIGN.md`
- Modify: `~/.claude/skills/formato-curso-v5/references/CHECKLIST-V5.md`

**Interfaces:** none (documentation only).

- [ ] **Step 1: `SKILL.md` — document `.termdemo`**

Using Edit, find this line (the icon table's closing, right before "Registro visual"):

```
**Repertório de metáforas candidatas** (adaptar ao tema do curso, nunca copiar verbatim entre cursos): estagiário novo que não conhece a empresa · molde de costura reutilizável · gaveta de ferramentas organizada · receita de bolo com variações · assistente que precisa de contexto para ajudar bem · caderno de receitas de família · placa de sinalização num caminho conhecido. Banco completo de analogias por profissão vive em `EXEMPLOS-PROFISSOES.md`.
```

That exact text lives in `V5-DESIGN.md`, not `SKILL.md` — this step instead edits `SKILL.md`. Using Edit on `SKILL.md`, find:

```
## Prática multi-modo
```

Replace with:

```
## Terceiro registro visual: `.termdemo` (demo real de comando)

Todo step já tem 1 slot de figura na rail (`data-fig`, ver contrato herdado do v4). `.termdemo`
é uma **segunda opção de conteúdo pra esse mesmo slot** — não é um componente novo sujeito ao
teto de 9 componentes + 3 quadros nem à regra de contenção AD-7. Use `.termdemo` (classe
`fig termdemo` na rail, `class="mobfig"` com cópia idêntica sem a classe `fig` no corpo do
step, pro mobile) **só quando o step tiver um comando real** — nos demais, use o `.fig` de
diagrama-de-mecanismo normalmente. Interação: clique-pra-revelar (`.termdemo-reveal`) mostra
primeiro o par `data-variant="ok"`; um `.termdemo-toggle` alterna pra `data-variant="erro"` —
só um visível por vez. **Use o par certo+erro só quando existir um erro comum real e
plausível** (mesmo critério do `.qerr`, `RETENCAO-V5.md` §4) — senão, omita o segundo
`.termdemo-cmd` e o botão de toggle, deixando só o comando certo.

## Prática multi-modo
```

- [ ] **Step 2: `SKILL.md` — document the exercícios mechanism**

Using Edit, find:

```
## Painel D-09 (jornada estendida)
```

Replace with:

```
## Refazer exercícios (por aula e pra trilha inteira)

Herdado do motor, ao lado da revisão de cartões (que já existia): (1) **por aula**, um botão
"refazer exercício desta aula" na mesma caixa de fim de aula onde já mora "revisar os cartões
desta aula" — zera o progresso da `.practice` daquela aula e rola até ela; (2) **pra trilha
inteira**, um botão novo na barra do topo ("exercícios") abre um painel listando toda
`.practice` do curso, com "ir até ela" (navega e fecha o painel) e "resetar" (individual, sem
"resetar tudo") por linha. Sem estado novo — reaproveita `state.aulas[k].tasks`.

## Painel D-09 (jornada estendida)
```

- [ ] **Step 3: `V5-DESIGN.md` — document the third visual register**

Using Edit, find:

```
**Zero itens da lista proibida de metáfora:** robô humanoide, cérebro-circuito, matrix-rain (chuva de código), aperto de mão humano-robô — são os clichês visuais de IA que não passam no "teste de metáfora" (um profissional de 50 anos se reconhece nessa imagem? não, se reconhece num estagiário, numa gaveta de ferramentas, num molde de costura).
```

Replace with:

```
**Zero itens da lista proibida de metáfora:** robô humanoide, cérebro-circuito, matrix-rain (chuva de código), aperto de mão humano-robô — são os clichês visuais de IA que não passam no "teste de metáfora" (um profissional de 50 anos se reconhece nessa imagem? não, se reconhece num estagiário, numa gaveta de ferramentas, num molde de costura).

**Terceiro registro — `.termdemo` (demonstração literal de comando):** nem metáfora-do-mundo-real, nem diagrama-de-mecanismo — é a tela real, o comando e a saída tal como aparecem no terminal do aluno. Vive no mesmo slot de figura por step (rail + cópia mobile), só nos steps com comando real, nunca misturado com os outros dois registros no mesmo step.
```

- [ ] **Step 4: `CHECKLIST-V5.md` — new verification item**

Using Edit, find:

```
## 6. `.gterm` — cobertura do glossário (item herdado do v4, auditoria explícita)
```

Replace with:

```
## 5.1. `.termdemo` — cobertura nos steps com comando real

Todo step com `<div class="pcodewrap">` ou um comando citado no corpo do texto deve ter
`.termdemo` (não `.fig` de diagrama) no mesmo `data-fig`, com cópia idêntica em `.mobfig`
(sem a classe `fig`). Verificação: `grep -c 'pcodewrap'` vs `grep -c 'class="fig termdemo"'`
por aula — todo step com `pcodewrap` deve ter um `.termdemo` correspondente, **exceto**
quando não existir erro comum plausível nem a demonstração fizer sentido isolada (julgamento
do autor, registrado como decisão, não omissão silenciosa).

## 6. `.gterm` — cobertura do glossário (item herdado do v4, auditoria explícita)
```

- [ ] **Step 5: Verify all three files**

Run:
```bash
grep -c "termdemo" ~/.claude/skills/formato-curso-v5/SKILL.md
grep -c "refazer exerc" ~/.claude/skills/formato-curso-v5/SKILL.md
grep -c "termdemo" ~/.claude/skills/formato-curso-v5/references/V5-DESIGN.md
grep -c "termdemo" ~/.claude/skills/formato-curso-v5/references/CHECKLIST-V5.md
```
Expected: all four commands return a number ≥ `1`.

- [ ] **Step 6: Commit and push — closes Part A**

```bash
cd ~/projetos/formato-curso-inema
git add formato-curso-v5/SKILL.md formato-curso-v5/references/V5-DESIGN.md formato-curso-v5/references/CHECKLIST-V5.md
git commit -m "docs(v5): document .termdemo and refazer-exercícios"
git push origin v5-enhancements
git log --oneline -10
```
Expected: commit succeeds, `git push` succeeds, `git log` shows all of Part A's commits (Tasks 1–9) leading up to HEAD.

---

## Part B — Retrofit `ssh-basico` (`~/projetos/ssh-basico/`)

### Task 10: Copy updated assets from the skill into the course

**Files:**
- Modify: `~/projetos/ssh-basico/assets/aula.css` (overwrite with skill's copy)
- Modify: `~/projetos/ssh-basico/assets/curso.js` (overwrite with skill's copy)

**Interfaces:**
- Consumes: Tasks 1–6 (skill's `aula.css`/`curso.js`, now updated).

- [ ] **Step 1: Copy the files**

```bash
cp ~/.claude/skills/formato-curso-v5/assets/aula.css ~/projetos/ssh-basico/assets/aula.css
cp ~/.claude/skills/formato-curso-v5/assets/curso.js ~/projetos/ssh-basico/assets/curso.js
```

- [ ] **Step 2: Verify**

```bash
diff ~/.claude/skills/formato-curso-v5/assets/aula.css ~/projetos/ssh-basico/assets/aula.css
diff ~/.claude/skills/formato-curso-v5/assets/curso.js ~/projetos/ssh-basico/assets/curso.js
```
Expected: no output from either `diff` (files identical).

---

### Task 11: Add "exercícios" bar button to `curso.html`

**Files:**
- Modify: `~/projetos/ssh-basico/curso.html`

**Interfaces:**
- Consumes: `#exbtn` wiring from Task 6 (now present in `curso.js`, copied by Task 10).

- [ ] **Step 1: Edit the bar**

Using Edit, find (this appears exactly once in `curso.html`):

```
  <button id="revbtn">revisar <span id="revn">0</span></button>
  <button id="jorbtn">jornada</button>
```

Replace with:

```
  <button id="revbtn">revisar <span id="revn">0</span></button>
  <button id="exbtn">exercícios</button>
  <button id="jorbtn">jornada</button>
```

- [ ] **Step 2: Verify**

Run: `grep -c 'id="exbtn"' ~/projetos/ssh-basico/curso.html`
Expected: `1`

---

### Task 12: Wrap `.recap-autor`/`.next-action` in `.fecho-wrap` (all 9 aulas, scripted)

**Files:**
- Modify: `~/projetos/ssh-basico/curso.html` (via Python script, not manual Edit — same block repeats 9 times identically in structure)

**Interfaces:** none.

- [ ] **Step 1: Run the wrapping script**

```bash
cd ~/projetos/ssh-basico
python3 - <<'PY'
import re
with open('curso.html', encoding='utf-8') as f:
    content = f.read()

pattern = re.compile(
    r'(<div class="recap-autor">.*?</div>\s*<div class="next-action">.*?</div>)(\s*)(<script type="application/json" id="cards-)',
    re.S
)
content2, n = pattern.subn(r'<div class="fecho-wrap">\n\1\n</div>\2\3', content)
print("wrapped:", n)
assert n == 9, "expected exactly 9 replacements, got %d" % n
with open('curso.html', 'w', encoding='utf-8') as f:
    f.write(content2)
PY
```
Expected output: `wrapped: 9` (the script's own `assert` will raise `AssertionError` and abort the write if the count is wrong — safe to run).

- [ ] **Step 2: Verify**

```bash
grep -c 'class="fecho-wrap"' curso.html
```
Expected: `9`

- [ ] **Step 3: Re-run the original density/structure checklist to confirm nothing broke**

```bash
grep -c 'class="promise"' curso.html
grep -c 'class="recap-autor"' curso.html
grep -c 'class="next-action"' curso.html
grep -c '<!-- COMPLETUDE aula-' curso.html
grep -c '\[AUSENTE\]' curso.html
```
Expected: `9`, `9`, `9`, `9`, `0` respectively (identical to the counts already verified when the course was first published — this task only adds wrapper `<div>`s, it must not change any of these counts).

---

### Task 13: `.termdemo` — Aula 2, Step 1 (`ssh -V` vs `ssh -v`)

**Files:**
- Modify: `~/projetos/ssh-basico/curso.html`

**Interfaces:** none (content authoring).

- [ ] **Step 1: Add the rail `.termdemo` (replaces the plain `.fig` at `data-fig="1"` in aula 2)**

Using Edit, find (aula 2's rail, first figure):

```
      <div class="fig on" data-fig="1" data-cap="<b>Fig.</b> o terminal pergunta ao sistema se o SSH já existe"><svg viewBox="0 0 220 200" fill="none" aria-hidden="true">
        <rect x="30" y="50" width="160" height="100" rx="10" style="stroke:var(--ink-soft)" stroke-width="2"/>
        <path d="M50 90 L170 90 M50 110 L140 110" style="stroke:var(--accent)" stroke-width="2"/>
      </svg></div>
```

Replace with:

```
      <div class="fig termdemo on" data-fig="1" data-cap="<b>Fig.</b> repare na diferença entre -V (maiúsculo, versão) e -v (minúsculo, outra coisa)">
        <button class="termdemo-reveal" type="button">ver o que acontece →</button>
        <div class="termdemo-body" hidden>
          <div class="termdemo-cmd on" data-variant="ok">
            <p class="termdemo-k">comando certo</p>
            <pre class="termdemo-in">$ ssh -V</pre>
            <pre class="termdemo-out">OpenSSH_for_Windows_9.5, LibreSSL 3.9.2</pre>
          </div>
          <div class="termdemo-cmd" data-variant="erro">
            <p class="termdemo-k">erro comum</p>
            <pre class="termdemo-in">$ ssh -v</pre>
            <pre class="termdemo-out termdemo-err">OpenSSH_for_Windows_9.5, LibreSSL 3.9.2
debug1: Reading configuration data ...
(-v minúsculo liga o modo detalhado — não é isto que confirma a versão)</pre>
          </div>
          <button class="termdemo-toggle" type="button">ver o erro comum</button>
        </div>
      </div>
```

- [ ] **Step 2: Add the mobile copy inside step 1's body**

Step 1 of aula 2 currently has no `.mobfig` at all. Using Edit, find:

```
      <div class="pcodewrap"><button class="pcopy" type="button">copiar</button><pre class="pcode">ssh -V</pre></div>
      <button class="readbtn" data-read="s1">marcar como lido</button>
```

Replace with:

```
      <div class="pcodewrap"><button class="pcopy" type="button">copiar</button><pre class="pcode">ssh -V</pre></div>
      <div class="mobfig">
        <div class="termdemo">
          <button class="termdemo-reveal" type="button">ver o que acontece →</button>
          <div class="termdemo-body" hidden>
            <div class="termdemo-cmd on" data-variant="ok">
              <p class="termdemo-k">comando certo</p>
              <pre class="termdemo-in">$ ssh -V</pre>
              <pre class="termdemo-out">OpenSSH_for_Windows_9.5, LibreSSL 3.9.2</pre>
            </div>
            <div class="termdemo-cmd" data-variant="erro">
              <p class="termdemo-k">erro comum</p>
              <pre class="termdemo-in">$ ssh -v</pre>
              <pre class="termdemo-out termdemo-err">OpenSSH_for_Windows_9.5, LibreSSL 3.9.2
debug1: Reading configuration data ...
(-v minúsculo liga o modo detalhado — não é isto que confirma a versão)</pre>
            </div>
            <button class="termdemo-toggle" type="button">ver o erro comum</button>
          </div>
        </div>
      </div>
      <button class="readbtn" data-read="s1">marcar como lido</button>
```

- [ ] **Step 3: Verify**

```bash
grep -c 'class="fig termdemo' ~/projetos/ssh-basico/curso.html
```
Expected: `1` (first of 10 across the whole retrofit — this count grows by 1 each task from here on; Task 20 checks the final total).

---

### Task 14: `.termdemo` — Aula 3, Steps 1 and 3

**Files:**
- Modify: `~/projetos/ssh-basico/curso.html`

- [ ] **Step 1: Step 1 rail figure (`pwd`/`ls` — single variant, no natural common error)**

Using Edit, find:

```
      <div class="fig on" data-fig="1" data-cap="<b>Fig.</b> pwd mostra onde você está; ls mostra o que existe ali"><svg viewBox="0 0 220 200" fill="none" aria-hidden="true">
        <rect x="70" y="70" width="80" height="60" rx="8" style="stroke:var(--accent)" stroke-width="2"/>
        <circle cx="90" cy="90" r="4" style="fill:var(--ink-soft)"/>
        <circle cx="110" cy="90" r="4" style="fill:var(--ink-soft)"/>
        <circle cx="130" cy="90" r="4" style="fill:var(--ink-soft)"/>
      </svg></div>
```

Replace with:

```
      <div class="fig termdemo on" data-fig="1" data-cap="<b>Fig.</b> pwd mostra onde você está; ls mostra o que existe ali">
        <button class="termdemo-reveal" type="button">ver o que acontece →</button>
        <div class="termdemo-body" hidden>
          <div class="termdemo-cmd on" data-variant="ok">
            <p class="termdemo-k">comando</p>
            <pre class="termdemo-in">$ pwd
$ ls</pre>
            <pre class="termdemo-out">/root
automacao_whatsapp  config.json  logs</pre>
          </div>
        </div>
      </div>
```

- [ ] **Step 2: Step 1 mobile copy**

Using Edit, find:

```
      <div class="pcodewrap"><button class="pcopy" type="button">copiar</button><pre class="pcode">pwd
ls</pre></div>
      <button class="readbtn" data-read="s1">marcar como lido</button>
```

Replace with:

```
      <div class="pcodewrap"><button class="pcopy" type="button">copiar</button><pre class="pcode">pwd
ls</pre></div>
      <div class="mobfig">
        <div class="termdemo">
          <button class="termdemo-reveal" type="button">ver o que acontece →</button>
          <div class="termdemo-body" hidden>
            <div class="termdemo-cmd on" data-variant="ok">
              <p class="termdemo-k">comando</p>
              <pre class="termdemo-in">$ pwd
$ ls</pre>
              <pre class="termdemo-out">/root
automacao_whatsapp  config.json  logs</pre>
            </div>
          </div>
        </div>
      </div>
      <button class="readbtn" data-read="s1">marcar como lido</button>
```

- [ ] **Step 3: Step 3 rail figure (`sudo apt update` — pair with "forgot sudo")**

Using Edit, find:

```
      <div class="fig" data-fig="3" data-cap="<b>Fig.</b> uname mostra o sistema; apt update busca as atualizações disponíveis"><svg viewBox="0 0 220 200" fill="none" aria-hidden="true">
        <rect x="60" y="60" width="100" height="80" rx="10" style="stroke:var(--accent)" stroke-width="2"/>
        <path d="M85 90 L135 90 M85 110 L120 110" style="stroke:var(--ink-soft)" stroke-width="2"/>
      </svg></div>
```

Replace with:

```
      <div class="fig termdemo" data-fig="3" data-cap="<b>Fig.</b> esquecer o sudo é o erro mais comum ao atualizar o sistema">
        <button class="termdemo-reveal" type="button">ver o que acontece →</button>
        <div class="termdemo-body" hidden>
          <div class="termdemo-cmd on" data-variant="ok">
            <p class="termdemo-k">comando certo</p>
            <pre class="termdemo-in">$ sudo apt update && sudo apt upgrade -y</pre>
            <pre class="termdemo-out">Lendo listas de pacotes... Pronto
Todos os pacotes estão atualizados.</pre>
          </div>
          <div class="termdemo-cmd" data-variant="erro">
            <p class="termdemo-k">erro comum</p>
            <pre class="termdemo-in">$ apt update</pre>
            <pre class="termdemo-out termdemo-err">Permission denied (você precisa de privilégio de administrador)
E: Não é possível abrir o arquivo de trava — esqueceu o "sudo" na frente</pre>
          </div>
          <button class="termdemo-toggle" type="button">ver o erro comum</button>
        </div>
      </div>
```

- [ ] **Step 4: Step 3 mobile copy**

Using Edit, find:

```
      <div class="pcodewrap"><button class="pcopy" type="button">copiar</button><pre class="pcode">uname -a
sudo apt update && sudo apt upgrade -y</pre></div>
      <div class="quiz" data-answer="b">
```

Replace with:

```
      <div class="pcodewrap"><button class="pcopy" type="button">copiar</button><pre class="pcode">uname -a
sudo apt update && sudo apt upgrade -y</pre></div>
      <div class="mobfig">
        <div class="termdemo">
          <button class="termdemo-reveal" type="button">ver o que acontece →</button>
          <div class="termdemo-body" hidden>
            <div class="termdemo-cmd on" data-variant="ok">
              <p class="termdemo-k">comando certo</p>
              <pre class="termdemo-in">$ sudo apt update && sudo apt upgrade -y</pre>
              <pre class="termdemo-out">Lendo listas de pacotes... Pronto
Todos os pacotes estão atualizados.</pre>
            </div>
            <div class="termdemo-cmd" data-variant="erro">
              <p class="termdemo-k">erro comum</p>
              <pre class="termdemo-in">$ apt update</pre>
              <pre class="termdemo-out termdemo-err">Permission denied (você precisa de privilégio de administrador)
E: Não é possível abrir o arquivo de trava — esqueceu o "sudo" na frente</pre>
            </div>
            <button class="termdemo-toggle" type="button">ver o erro comum</button>
          </div>
        </div>
      </div>
      <div class="quiz" data-answer="b">
```

- [ ] **Step 5: Verify**

```bash
grep -c 'class="fig termdemo' ~/projetos/ssh-basico/curso.html
```
Expected: `3`

---

### Task 15: `.termdemo` — Aula 6, Steps 1 and 3

**Files:**
- Modify: `~/projetos/ssh-basico/curso.html`

- [ ] **Step 1: Step 1 rail figure (`type ...id_ed25519.pub` — pair with wrong filename)**

Using Edit, find:

```
      <div class="fig on" data-fig="1" data-cap="<b>Fig.</b> a chave pública é um texto legível, sempre começando por ssh-ed25519"><svg viewBox="0 0 220 200" fill="none" aria-hidden="true">
        <rect x="40" y="80" width="140" height="40" rx="8" style="stroke:var(--accent)" stroke-width="2"/>
        <path d="M55 100 L165 100" style="stroke:var(--ink-soft)" stroke-width="1.5" stroke-dasharray="3 3"/>
      </svg></div>
```

Replace with:

```
      <div class="fig termdemo on" data-fig="1" data-cap="<b>Fig.</b> a chave pública é um texto legível, sempre começando por ssh-ed25519">
        <button class="termdemo-reveal" type="button">ver o que acontece →</button>
        <div class="termdemo-body" hidden>
          <div class="termdemo-cmd on" data-variant="ok">
            <p class="termdemo-k">comando certo</p>
            <pre class="termdemo-in">$ type %USERPROFILE%\.ssh\id_ed25519.pub</pre>
            <pre class="termdemo-out">ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... minha-vps</pre>
          </div>
          <div class="termdemo-cmd" data-variant="erro">
            <p class="termdemo-k">erro comum</p>
            <pre class="termdemo-in">$ type %USERPROFILE%\.ssh\minha_vps.pub</pre>
            <pre class="termdemo-out termdemo-err">O sistema não pode encontrar o arquivo especificado.
(nome de arquivo errado — confira o nome exato com dir antes)</pre>
          </div>
          <button class="termdemo-toggle" type="button">ver o erro comum</button>
        </div>
      </div>
```

- [ ] **Step 2: Step 1 mobile copy**

Using Edit, find:

```
      <div class="pcodewrap"><button class="pcopy" type="button">copiar</button><pre class="pcode">type %USERPROFILE%\.ssh\id_ed25519.pub</pre></div>
      <button class="readbtn" data-read="s1">marcar como lido</button>
```

Replace with:

```
      <div class="pcodewrap"><button class="pcopy" type="button">copiar</button><pre class="pcode">type %USERPROFILE%\.ssh\id_ed25519.pub</pre></div>
      <div class="mobfig">
        <div class="termdemo">
          <button class="termdemo-reveal" type="button">ver o que acontece →</button>
          <div class="termdemo-body" hidden>
            <div class="termdemo-cmd on" data-variant="ok">
              <p class="termdemo-k">comando certo</p>
              <pre class="termdemo-in">$ type %USERPROFILE%\.ssh\id_ed25519.pub</pre>
              <pre class="termdemo-out">ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAA... minha-vps</pre>
            </div>
            <div class="termdemo-cmd" data-variant="erro">
              <p class="termdemo-k">erro comum</p>
              <pre class="termdemo-in">$ type %USERPROFILE%\.ssh\minha_vps.pub</pre>
              <pre class="termdemo-out termdemo-err">O sistema não pode encontrar o arquivo especificado.
(nome de arquivo errado — confira o nome exato com dir antes)</pre>
            </div>
            <button class="termdemo-toggle" type="button">ver o erro comum</button>
          </div>
        </div>
      </div>
      <button class="readbtn" data-read="s1">marcar como lido</button>
```

- [ ] **Step 3: Step 3 rail figure (`ssh-keygen` — pair with running it twice, overwrite prompt)**

Using Edit, find:

```
      <div class="fig" data-fig="3" data-cap="<b>Fig.</b> ssh-keygen fabrica um par de chaves novo, sob medida"><svg viewBox="0 0 220 200" fill="none" aria-hidden="true">
        <circle cx="110" cy="100" r="35" style="stroke:var(--accent)" stroke-width="2"/>
        <path d="M110 65 L110 40" style="stroke:var(--accent)" stroke-width="2"/>
      </svg></div>
```

Replace with:

```
      <div class="fig termdemo" data-fig="3" data-cap="<b>Fig.</b> rodar ssh-keygen 2x com o mesmo nome pergunta antes de sobrescrever">
        <button class="termdemo-reveal" type="button">ver o que acontece →</button>
        <div class="termdemo-body" hidden>
          <div class="termdemo-cmd on" data-variant="ok">
            <p class="termdemo-k">comando certo</p>
            <pre class="termdemo-in">$ ssh-keygen -t ed25519 -C "minha-vps" -f %USERPROFILE%\.ssh\minha_vps</pre>
            <pre class="termdemo-out">Generating public/private ed25519 key pair.
Your identification has been saved in minha_vps
Your public key has been saved in minha_vps.pub</pre>
          </div>
          <div class="termdemo-cmd" data-variant="erro">
            <p class="termdemo-k">erro comum</p>
            <pre class="termdemo-in">$ ssh-keygen -t ed25519 -C "minha-vps" -f %USERPROFILE%\.ssh\minha_vps</pre>
            <pre class="termdemo-out termdemo-err">minha_vps already exists.
Overwrite (y/n)?
(já existe uma chave com esse nome — escolha outro nome em vez de sobrescrever sem querer)</pre>
          </div>
          <button class="termdemo-toggle" type="button">ver o erro comum</button>
        </div>
      </div>
```

- [ ] **Step 4: Step 3 mobile copy**

Using Edit, find:

```
      <div class="pcodewrap"><button class="pcopy" type="button">copiar</button><pre class="pcode">ssh-keygen -t ed25519 -C "minha-vps" -f %USERPROFILE%\.ssh\minha_vps</pre></div>
      <p>Depois disso, é só apertar Enter nas perguntas seguintes (elas pedem uma senha extra opcional pra chave — pode deixar em branco e confirmar).</p>
```

Replace with:

```
      <div class="pcodewrap"><button class="pcopy" type="button">copiar</button><pre class="pcode">ssh-keygen -t ed25519 -C "minha-vps" -f %USERPROFILE%\.ssh\minha_vps</pre></div>
      <div class="mobfig">
        <div class="termdemo">
          <button class="termdemo-reveal" type="button">ver o que acontece →</button>
          <div class="termdemo-body" hidden>
            <div class="termdemo-cmd on" data-variant="ok">
              <p class="termdemo-k">comando certo</p>
              <pre class="termdemo-in">$ ssh-keygen -t ed25519 -C "minha-vps" -f %USERPROFILE%\.ssh\minha_vps</pre>
              <pre class="termdemo-out">Generating public/private ed25519 key pair.
Your identification has been saved in minha_vps
Your public key has been saved in minha_vps.pub</pre>
            </div>
            <div class="termdemo-cmd" data-variant="erro">
              <p class="termdemo-k">erro comum</p>
              <pre class="termdemo-in">$ ssh-keygen -t ed25519 -C "minha-vps" -f %USERPROFILE%\.ssh\minha_vps</pre>
              <pre class="termdemo-out termdemo-err">minha_vps already exists.
Overwrite (y/n)?
(já existe uma chave com esse nome — escolha outro nome em vez de sobrescrever sem querer)</pre>
            </div>
            <button class="termdemo-toggle" type="button">ver o erro comum</button>
          </div>
        </div>
      </div>
      <p>Depois disso, é só apertar Enter nas perguntas seguintes (elas pedem uma senha extra opcional pra chave — pode deixar em branco e confirmar).</p>
```

- [ ] **Step 5: Verify**

```bash
grep -c 'class="fig termdemo' ~/projetos/ssh-basico/curso.html
```
Expected: `5`

---

### Task 16: `.termdemo` — Aula 7, Steps 2, 3, 4

**Files:**
- Modify: `~/projetos/ssh-basico/curso.html`

- [ ] **Step 1: Step 2 rail figure (`mkdir`/`nano` — single variant; the failure only shows up later in step 3/4)**

Using Edit, find:

```
      <div class="fig" data-fig="2" data-cap="<b>Fig.</b> a pasta .ssh e o authorized_keys, criados dentro da própria VPS"><svg viewBox="0 0 220 200" fill="none" aria-hidden="true">
        <rect x="60" y="50" width="100" height="100" rx="10" style="stroke:var(--accent)" stroke-width="2"/>
        <rect x="80" y="90" width="60" height="30" rx="6" style="stroke:var(--ink-soft)" stroke-width="2"/>
      </svg></div>
```

Replace with:

```
      <div class="fig termdemo" data-fig="2" data-cap="<b>Fig.</b> mkdir cria a pasta; nano abre o arquivo pra colar a chave">
        <button class="termdemo-reveal" type="button">ver o que acontece →</button>
        <div class="termdemo-body" hidden>
          <div class="termdemo-cmd on" data-variant="ok">
            <p class="termdemo-k">comando</p>
            <pre class="termdemo-in">$ mkdir -p ~/.ssh
$ nano ~/.ssh/authorized_keys</pre>
            <pre class="termdemo-out">(o nano abre uma tela de edição — cole a chave, CTRL+O, Enter, CTRL+X)</pre>
          </div>
        </div>
      </div>
```

- [ ] **Step 2: Step 2 mobile copy (this step already has `.mobfig` — replace its contents)**

Using Edit, find (note this step already has `data-mobfig="2"`, an empty placeholder):

```
      <div class="mobfig" data-mobfig="2"></div>
      <button class="readbtn" data-read="s2">marcar como lido</button>
```

This exact snippet `<div class="mobfig" data-mobfig="2"></div>` appears more than once in the file (each aula reuses this pattern) — to target aula 7 specifically, search within context. Using Edit with enough surrounding context to be unique:

Find:

```
      <p>O <code>nano</code> é um editor de texto simples que abre dentro do próprio terminal. Cole ali a chave pública copiada, e salve com <code>CTRL+O</code>, <code>Enter</code>, depois <code>CTRL+X</code> pra sair. Foi assim que Ricardo colou a chave pública da loja dentro da VPS, na primeira vez que fez esse passo.</p>
      <div class="qsteps">
        <p class="qk">Checklist completo: chave no seu PC até dentro da VPS</p>
        <ol>
          <li>Ver e copiar a chave pública no seu computador.</li>
          <li>Entrar na VPS com senha.</li>
          <li><code>mkdir -p ~/.ssh</code> — criar a pasta, se ainda não existir.</li>
          <li><code>nano ~/.ssh/authorized_keys</code> — colar a chave e salvar (<code>CTRL+O</code>, Enter, <code>CTRL+X</code>).</li>
        </ol>
      </div>
      <div class="mobfig" data-mobfig="2"></div>
      <button class="readbtn" data-read="s2">marcar como lido</button>
```

Replace with:

```
      <p>O <code>nano</code> é um editor de texto simples que abre dentro do próprio terminal. Cole ali a chave pública copiada, e salve com <code>CTRL+O</code>, <code>Enter</code>, depois <code>CTRL+X</code> pra sair. Foi assim que Ricardo colou a chave pública da loja dentro da VPS, na primeira vez que fez esse passo.</p>
      <div class="qsteps">
        <p class="qk">Checklist completo: chave no seu PC até dentro da VPS</p>
        <ol>
          <li>Ver e copiar a chave pública no seu computador.</li>
          <li>Entrar na VPS com senha.</li>
          <li><code>mkdir -p ~/.ssh</code> — criar a pasta, se ainda não existir.</li>
          <li><code>nano ~/.ssh/authorized_keys</code> — colar a chave e salvar (<code>CTRL+O</code>, Enter, <code>CTRL+X</code>).</li>
        </ol>
      </div>
      <div class="mobfig">
        <div class="termdemo">
          <button class="termdemo-reveal" type="button">ver o que acontece →</button>
          <div class="termdemo-body" hidden>
            <div class="termdemo-cmd on" data-variant="ok">
              <p class="termdemo-k">comando</p>
              <pre class="termdemo-in">$ mkdir -p ~/.ssh
$ nano ~/.ssh/authorized_keys</pre>
              <pre class="termdemo-out">(o nano abre uma tela de edição — cole a chave, CTRL+O, Enter, CTRL+X)</pre>
            </div>
          </div>
        </div>
      </div>
      <button class="readbtn" data-read="s2">marcar como lido</button>
```

- [ ] **Step 3: Step 3 rail figure (`chmod` — pair with skipping it, Permission denied later)**

Using Edit, find:

```
      <div class="fig" data-fig="3" data-cap="<b>Fig.</b> chmod ajusta quem pode ler ou escrever na pasta e no arquivo"><svg viewBox="0 0 220 200" fill="none" aria-hidden="true">
        <rect x="70" y="70" width="80" height="60" rx="8" style="stroke:var(--accent)" stroke-width="2"/>
        <path d="M95 100 L145 100" style="stroke:var(--ink-soft)" stroke-width="2"/>
        <circle cx="150" cy="100" r="5" style="stroke:var(--accent)" stroke-width="2"/>
      </svg></div>
```

Replace with:

```
      <div class="fig termdemo" data-fig="3" data-cap="<b>Fig.</b> chmod não mostra nada quando dá certo — o silêncio é o sinal de sucesso">
        <button class="termdemo-reveal" type="button">ver o que acontece →</button>
        <div class="termdemo-body" hidden>
          <div class="termdemo-cmd on" data-variant="ok">
            <p class="termdemo-k">comando certo</p>
            <pre class="termdemo-in">$ chmod 700 ~/.ssh
$ chmod 600 ~/.ssh/authorized_keys</pre>
            <pre class="termdemo-out">(nada aparece — no Linux, silêncio depois de um comando quase sempre significa sucesso)</pre>
          </div>
          <div class="termdemo-cmd" data-variant="erro">
            <p class="termdemo-k">erro comum</p>
            <pre class="termdemo-in">$ ssh -i minha_vps root@IP_DA_VPS
(pulando o chmod)</pre>
            <pre class="termdemo-out termdemo-err">root@IP: Permission denied (publickey,password).
(a VPS recusa a chave porque as permissões da pasta .ssh estão abertas demais)</pre>
          </div>
          <button class="termdemo-toggle" type="button">ver o erro comum</button>
        </div>
      </div>
```

- [ ] **Step 4: Step 3 mobile copy**

Using Edit, find:

```
      <p>Ricardo esqueceu esse passo na primeira tentativa e sua chave simplesmente não funcionou — sem nenhuma mensagem de erro clara. Depois de ajustar as permissões, funcionou de primeira.</p>
      <div class="quiz" data-answer="b">
```

Replace with:

```
      <p>Ricardo esqueceu esse passo na primeira tentativa e sua chave simplesmente não funcionou — sem nenhuma mensagem de erro clara. Depois de ajustar as permissões, funcionou de primeira.</p>
      <div class="mobfig">
        <div class="termdemo">
          <button class="termdemo-reveal" type="button">ver o que acontece →</button>
          <div class="termdemo-body" hidden>
            <div class="termdemo-cmd on" data-variant="ok">
              <p class="termdemo-k">comando certo</p>
              <pre class="termdemo-in">$ chmod 700 ~/.ssh
$ chmod 600 ~/.ssh/authorized_keys</pre>
              <pre class="termdemo-out">(nada aparece — no Linux, silêncio depois de um comando quase sempre significa sucesso)</pre>
            </div>
            <div class="termdemo-cmd" data-variant="erro">
              <p class="termdemo-k">erro comum</p>
              <pre class="termdemo-in">$ ssh -i minha_vps root@IP_DA_VPS
(pulando o chmod)</pre>
              <pre class="termdemo-out termdemo-err">root@IP: Permission denied (publickey,password).
(a VPS recusa a chave porque as permissões da pasta .ssh estão abertas demais)</pre>
            </div>
            <button class="termdemo-toggle" type="button">ver o erro comum</button>
          </div>
        </div>
      </div>
      <div class="quiz" data-answer="b">
```

- [ ] **Step 5: Step 4 rail figure (`ssh -i` final connection — pair with still being asked for a password)**

Using Edit, find:

```
      <div class="fig" data-fig="4" data-cap="<b>Fig.</b> conexão com -i: a chave apresentada, sem nenhuma senha pedida"><svg viewBox="0 0 220 200" fill="none" aria-hidden="true">
        <rect x="30" y="80" width="55" height="40" rx="6" style="stroke:var(--ink-soft)" stroke-width="2"/>
        <rect x="140" y="80" width="55" height="40" rx="6" style="stroke:var(--accent)" stroke-width="2"/>
        <path d="M85 100 L140 100" style="stroke:var(--accent)" stroke-width="2"/>
        <path d="M128 92 L140 100 L128 108" style="stroke:var(--accent)" stroke-width="2" fill="none"/>
      </svg></div>
```

Replace with:

```
      <div class="fig termdemo" data-fig="4" data-cap="<b>Fig.</b> com a chave certa, não aparece nenhum pedido de senha">
        <button class="termdemo-reveal" type="button">ver o que acontece →</button>
        <div class="termdemo-body" hidden>
          <div class="termdemo-cmd on" data-variant="ok">
            <p class="termdemo-k">comando certo</p>
            <pre class="termdemo-in">$ ssh -i %USERPROFILE%\.ssh\minha_vps root@123.123.123.123</pre>
            <pre class="termdemo-out">Welcome to Ubuntu 22.04 LTS
root@vps:~#</pre>
          </div>
          <div class="termdemo-cmd" data-variant="erro">
            <p class="termdemo-k">erro comum</p>
            <pre class="termdemo-in">$ ssh -i %USERPROFILE%\.ssh\minha_vps root@123.123.123.123</pre>
            <pre class="termdemo-out termdemo-err">root@123.123.123.123's password:
(ainda pediu senha — a chave não foi aceita: revise o authorized_keys e o chmod)</pre>
          </div>
          <button class="termdemo-toggle" type="button">ver o erro comum</button>
        </div>
      </div>
```

- [ ] **Step 6: Step 4 mobile copy (this step already has `.mobfig` — replace its contents)**

Confirmed exact current content at this location (`curso.html` lines 1155–1161):

```
      <p>Se tudo estiver certo, Marta entra direto, sem nenhuma tela pedindo senha — só a chave já basta.</p>
      <div class="qbefore-after">
        <div class="qba-col"><p class="qk">Antes</p><p>Toda conexão pedia a senha, e ela precisava lembrar ou copiar de algum lugar.</p></div>
        <div class="qba-col"><p class="qk">Depois</p><p>A chave prova quem ela é sozinha — conexão mais rápida e mais segura, sem senha nenhuma.</p></div>
      </div>
      <div class="mobfig" data-mobfig="4"></div>
      <button class="readbtn" data-read="s4">marcar como lido</button>
```

Using Edit, find exactly that block. Replace with:

```
      <p>Se tudo estiver certo, Marta entra direto, sem nenhuma tela pedindo senha — só a chave já basta.</p>
      <div class="qbefore-after">
        <div class="qba-col"><p class="qk">Antes</p><p>Toda conexão pedia a senha, e ela precisava lembrar ou copiar de algum lugar.</p></div>
        <div class="qba-col"><p class="qk">Depois</p><p>A chave prova quem ela é sozinha — conexão mais rápida e mais segura, sem senha nenhuma.</p></div>
      </div>
      <div class="mobfig">
        <div class="termdemo">
          <button class="termdemo-reveal" type="button">ver o que acontece →</button>
          <div class="termdemo-body" hidden>
            <div class="termdemo-cmd on" data-variant="ok">
              <p class="termdemo-k">comando certo</p>
              <pre class="termdemo-in">$ ssh -i %USERPROFILE%\.ssh\minha_vps root@123.123.123.123</pre>
              <pre class="termdemo-out">Welcome to Ubuntu 22.04 LTS
root@vps:~#</pre>
            </div>
            <div class="termdemo-cmd" data-variant="erro">
              <p class="termdemo-k">erro comum</p>
              <pre class="termdemo-in">$ ssh -i %USERPROFILE%\.ssh\minha_vps root@123.123.123.123</pre>
              <pre class="termdemo-out termdemo-err">root@123.123.123.123's password:
(ainda pediu senha — a chave não foi aceita: revise o authorized_keys e o chmod)</pre>
            </div>
            <button class="termdemo-toggle" type="button">ver o erro comum</button>
          </div>
        </div>
      </div>
      <button class="readbtn" data-read="s4">marcar como lido</button>
```

- [ ] **Step 7: Verify**

```bash
grep -c 'class="fig termdemo' ~/projetos/ssh-basico/curso.html
```
Expected: `8`

---

### Task 17: `.termdemo` — Aula 8, Step 2

**Files:**
- Modify: `~/projetos/ssh-basico/curso.html`

- [ ] **Step 1: Rail figure (config file → `ssh <apelido>` outcome, pair with bad indentation)**

Using Edit, find:

```
      <div class="fig" data-fig="2" data-cap="<b>Fig.</b> um bloco Host/HostName/User/IdentityFile dentro do arquivo config"><svg viewBox="0 0 220 200" fill="none" aria-hidden="true">
        <rect x="50" y="50" width="120" height="100" rx="10" style="stroke:var(--accent)" stroke-width="2"/>
        <path d="M70 75 L150 75 M80 95 L140 95 M80 115 L150 115 M80 135 L140 135" style="stroke:var(--ink-soft)" stroke-width="1.5"/>
      </svg></div>
```

Replace with:

```
      <div class="fig termdemo" data-fig="2" data-cap="<b>Fig.</b> config bem recuado conecta pelo apelido; mal recuado, não reconhece o host">
        <button class="termdemo-reveal" type="button">ver o que acontece →</button>
        <div class="termdemo-body" hidden>
          <div class="termdemo-cmd on" data-variant="ok">
            <p class="termdemo-k">comando certo</p>
            <pre class="termdemo-in">$ ssh hostinger</pre>
            <pre class="termdemo-out">Welcome to Ubuntu 22.04 LTS
root@vps:~#</pre>
          </div>
          <div class="termdemo-cmd" data-variant="erro">
            <p class="termdemo-k">erro comum</p>
            <pre class="termdemo-in">$ ssh hostinger</pre>
            <pre class="termdemo-out termdemo-err">ssh: Could not resolve hostname hostinger: Name or service not known
(o arquivo config existe, mas as linhas abaixo de "Host" não estavam recuadas)</pre>
          </div>
          <button class="termdemo-toggle" type="button">ver o erro comum</button>
        </div>
      </div>
```

- [ ] **Step 2: Mobile copy (this step already has `.mobfig` — replace its contents)**

Using Edit, find:

```
      <div class="qerr">
        <p class="qk">Erro comum</p>
        <p class="qerr-txt"><b>Esquecer o recuo (espaços) antes de cada linha depois de "Host".</b> O arquivo config segue um formato onde as linhas de baixo do "Host" precisam estar recuadas — sem isso, a conexão pelo apelido não funciona.</p>
      </div>
      <div class="mobfig" data-mobfig="2"></div>
      <button class="readbtn" data-read="s2">marcar como lido</button>
```

Replace with:

```
      <div class="qerr">
        <p class="qk">Erro comum</p>
        <p class="qerr-txt"><b>Esquecer o recuo (espaços) antes de cada linha depois de "Host".</b> O arquivo config segue um formato onde as linhas de baixo do "Host" precisam estar recuadas — sem isso, a conexão pelo apelido não funciona.</p>
      </div>
      <div class="mobfig">
        <div class="termdemo">
          <button class="termdemo-reveal" type="button">ver o que acontece →</button>
          <div class="termdemo-body" hidden>
            <div class="termdemo-cmd on" data-variant="ok">
              <p class="termdemo-k">comando certo</p>
              <pre class="termdemo-in">$ ssh hostinger</pre>
              <pre class="termdemo-out">Welcome to Ubuntu 22.04 LTS
root@vps:~#</pre>
            </div>
            <div class="termdemo-cmd" data-variant="erro">
              <p class="termdemo-k">erro comum</p>
              <pre class="termdemo-in">$ ssh hostinger</pre>
              <pre class="termdemo-out termdemo-err">ssh: Could not resolve hostname hostinger: Name or service not known
(o arquivo config existe, mas as linhas abaixo de "Host" não estavam recuadas)</pre>
            </div>
            <button class="termdemo-toggle" type="button">ver o erro comum</button>
          </div>
        </div>
      </div>
      <button class="readbtn" data-read="s2">marcar como lido</button>
```

- [ ] **Step 3: Verify**

```bash
grep -c 'class="fig termdemo' ~/projetos/ssh-basico/curso.html
```
Expected: `9`

---

### Task 18: `.termdemo` — Aula 9, Step 4

**Files:**
- Modify: `~/projetos/ssh-basico/curso.html`

- [ ] **Step 1: Rail figure (`systemctl status ssh` vs `sshd` — pair with wrong service name)**

Using Edit, find:

```
      <div class="fig" data-fig="4" data-cap="<b>Fig.</b> systemctl status confirma, de dentro da VPS, se o serviço está de pé"><svg viewBox="0 0 220 200" fill="none" aria-hidden="true">
        <rect x="70" y="60" width="80" height="80" rx="10" style="stroke:var(--accent)" stroke-width="2"/>
        <circle cx="110" cy="100" r="15" style="stroke:var(--accent)" stroke-width="2"/>
      </svg></div>
```

Replace with:

```
      <div class="fig termdemo" data-fig="4" data-cap="<b>Fig.</b> o nome certo do serviço muda de sistema pra sistema — ssh ou sshd">
        <button class="termdemo-reveal" type="button">ver o que acontece →</button>
        <div class="termdemo-body" hidden>
          <div class="termdemo-cmd on" data-variant="ok">
            <p class="termdemo-k">comando certo</p>
            <pre class="termdemo-in">$ systemctl status ssh</pre>
            <pre class="termdemo-out">● ssh.service - OpenBSD Secure Shell server
     Active: active (running)</pre>
          </div>
          <div class="termdemo-cmd" data-variant="erro">
            <p class="termdemo-k">erro comum</p>
            <pre class="termdemo-in">$ systemctl status sshd</pre>
            <pre class="termdemo-out termdemo-err">Unit sshd.service could not be found.
(neste sistema o serviço se chama "ssh", não "sshd" — tente o outro nome)</pre>
          </div>
          <button class="termdemo-toggle" type="button">ver o erro comum</button>
        </div>
      </div>
```

- [ ] **Step 2: Mobile copy (this step already has `.mobfig` — replace its contents)**

Using Edit, find:

```
      <div class="pcodewrap"><button class="pcopy" type="button">copiar</button><pre class="pcode">systemctl status ssh
sudo systemctl enable ssh
sudo systemctl start ssh</pre></div>
      <div class="qsteps">
```

Replace with:

```
      <div class="pcodewrap"><button class="pcopy" type="button">copiar</button><pre class="pcode">systemctl status ssh
sudo systemctl enable ssh
sudo systemctl start ssh</pre></div>
      <div class="mobfig">
        <div class="termdemo">
          <button class="termdemo-reveal" type="button">ver o que acontece →</button>
          <div class="termdemo-body" hidden>
            <div class="termdemo-cmd on" data-variant="ok">
              <p class="termdemo-k">comando certo</p>
              <pre class="termdemo-in">$ systemctl status ssh</pre>
              <pre class="termdemo-out">● ssh.service - OpenBSD Secure Shell server
     Active: active (running)</pre>
            </div>
            <div class="termdemo-cmd" data-variant="erro">
              <p class="termdemo-k">erro comum</p>
              <pre class="termdemo-in">$ systemctl status sshd</pre>
              <pre class="termdemo-out termdemo-err">Unit sshd.service could not be found.
(neste sistema o serviço se chama "ssh", não "sshd" — tente o outro nome)</pre>
            </div>
            <button class="termdemo-toggle" type="button">ver o erro comum</button>
          </div>
        </div>
      </div>
      <div class="qsteps">
```

- [ ] **Step 3: Verify — this completes all 10 `.termdemo` instances**

```bash
grep -c 'class="fig termdemo' ~/projetos/ssh-basico/curso.html
```
Expected: `10`

```bash
grep -c 'class="termdemo"' ~/projetos/ssh-basico/curso.html
```
Expected: `10` (the 10 mobile copies — each is `class="termdemo"` alone, no `fig`).

---

### Task 19: `landing.html` header — add course name

**Files:**
- Modify: `~/projetos/ssh-basico/landing.html`

- [ ] **Step 1: Add the CSS rule**

Using Edit, find:

```
  header.top .pro{background:var(--accent);color:var(--accent-ink);border-radius:4px;padding:2px 6px;font-size:9.5px;font-weight:700;letter-spacing:.1em;text-decoration:none;margin-left:8px}
```

Replace with:

```
  header.top .pro{background:var(--accent);color:var(--accent-ink);border-radius:4px;padding:2px 6px;font-size:9.5px;font-weight:700;letter-spacing:.1em;text-decoration:none;margin-left:8px}
  header.top .course{color:var(--muted);font-size:12px;letter-spacing:.04em;border-left:1px solid var(--line2);padding-left:10px;margin-left:2px;font-family:var(--mono)}
```

- [ ] **Step 2: Add the markup**

Using Edit, find:

```
<header class="top"><div class="wrap">
  <a class="mark" href="https://inema.club" target="_blank" rel="noopener">INEMA.CLUB</a>
  <a class="pro" href="https://inema.pro" target="_blank" rel="noopener">PRO</a>
</div></header>
```

Replace with:

```
<header class="top"><div class="wrap">
  <a class="mark" href="https://inema.club" target="_blank" rel="noopener">INEMA.CLUB</a>
  <a class="pro" href="https://inema.pro" target="_blank" rel="noopener">PRO</a>
  <span class="course">SSH e Chaves SSH</span>
</div></header>
```

- [ ] **Step 3: Verify**

```bash
grep -c 'class="course"' ~/projetos/ssh-basico/landing.html
```
Expected: `1`

---

### Task 20: Final verification, manual browser check, commit, push

**Files:** none modified (verification + git only).

- [ ] **Step 1: Re-run the full original checklist-portão (must still all pass after the retrofit)**

```bash
cd ~/projetos/ssh-basico
echo "promise: $(grep -c 'class="promise"' curso.html)"
echo "tempo: $(grep -oc 'data-tempo="[^"]*"' curso.html)"
echo "why: $(grep -c 'class="why"' curso.html)"
echo "practice: $(grep -c 'class="practice"' curso.html)"
echo "psafe: $(grep -c 'class="psafe"' curso.html)"
echo "recap-autor: $(grep -c 'class="recap-autor"' curso.html)"
echo "next-action: $(grep -c 'class="next-action"' curso.html)"
echo "fecho-wrap: $(grep -c 'class="fecho-wrap"' curso.html)"
echo "COMPLETUDE: $(grep -c '<!-- COMPLETUDE aula-' curso.html)"
echo "AUSENTE (must be 0): $(grep -c '\[AUSENTE\]' curso.html)"
echo "termdemo rail (must be 10): $(grep -c 'class="fig termdemo' curso.html)"
echo "termdemo mobile (must be 10): $(grep -c 'class="termdemo"' curso.html)"
echo "exbtn: $(grep -c 'id="exbtn"' curso.html)"
```
Expected: `promise/tempo/why/practice/psafe/recap-autor/next-action/COMPLETUDE` all `9`; `fecho-wrap` `9`; `AUSENTE` `0`; `termdemo rail` `10`; `termdemo mobile` `10`; `exbtn` `1`.

- [ ] **Step 2: Re-run the anti-duplication + density checks from the original publish (they must be unaffected — this retrofit did not touch recap/cards content)**

```bash
python3 - <<'PY'
import re, json
with open('curso.html', encoding='utf-8') as f:
    content = f.read()
def strip_tags(s): return re.sub(r'<[^>]+>', ' ', s)
def words(s): return re.findall(r"[a-zà-ÿ0-9']+", strip_tags(s).lower())
sections = re.split(r'(?=<section class="view" id="v-aula-)', content)
ok = True
for sec in sections:
    m = re.match(r'<section class="view" id="v-aula-(\d+)"', sec)
    if not m: continue
    aula = m.group(1)
    recap_m = re.search(r'<div class="recap-autor">(.*?)</div>', sec, re.S)
    recap_words = words(recap_m.group(1) if recap_m else '')
    recap_set6 = set(tuple(recap_words[i:i+6]) for i in range(max(0,len(recap_words)-5)))
    cards_m = re.search(r'<script type="application/json" id="cards-\d+">(.*?)</script>', sec, re.S)
    cards = json.loads(cards_m.group(1)) if cards_m else []
    overlap=False
    for c in cards:
        for field in ('front','back'):
            w = words(c[field])
            for i in range(max(0,len(w)-5)):
                if tuple(w[i:i+6]) in recap_set6: overlap=True
    if overlap: ok=False
    print(f"Aula {aula}: overlap={overlap}")
print("ALL OK" if ok else "FAILURE")
PY
```
Expected: `overlap=False` for all 9 aulas, `ALL OK` at the end.

- [ ] **Step 3: JS syntax check on the final course copy**

```bash
node --check ~/projetos/ssh-basico/assets/curso.js
```
Expected: no output, exit code 0.

- [ ] **Step 4: Manual browser smoke test**

Open `~/projetos/ssh-basico/curso.html` directly in a browser (`file://` path — the course is designed to work offline) and manually confirm:
- Aula 2, step 1: the rail shows a "ver o que acontece →" button instead of an SVG diagram; clicking it reveals the `ssh -V` output; clicking "ver o erro comum" swaps to the `ssh -v` output and the button now reads "ver o certo".
- Scroll to the bottom of aula 2: two buttons appear ("revisar os cartões desta aula" and "refazer exercício desta aula"); the recap text and next-action box now visually line up with the steps above (not stretched edge-to-edge).
- Top bar: a new "exercícios" button appears between "revisar" and "jornada"; clicking it opens a panel listing all 9 aulas with "ir até ela" / "resetar" per row.
- Resize the browser to a narrow (phone-width) viewport on aula 2: the mobile card shows the same click-to-reveal termdemo (not empty).
- Open `~/projetos/ssh-basico/landing.html`: the header now shows "SSH e Chaves SSH" next to the PRO badge.

If any of these fail, stop and fix before committing — this is the only end-to-end check available (no automated browser test exists in this project).

- [ ] **Step 5: Commit and push**

```bash
cd ~/projetos/ssh-basico
git add assets/aula.css assets/curso.js curso.html landing.html
git status
git commit -m "$(cat <<'EOF'
Add termdemo component, exercise-review mechanism, fix recap alignment

Retrofits the ssh-basico course with 3 skill-level improvements found
during review: a real command demonstration (.termdemo, click-to-reveal
pairs of correct command + common mistake) replacing plain-prose
examples in the 6 aulas with real terminal commands; a "refazer
exercícios" mechanism (per-aula button + whole-trilha panel) alongside
the existing flashcard review; and a CSS fix so the end-of-aula recap
aligns with the reading column instead of stretching full-width.

Same changes were made first in the formato-curso-v5 skill itself
(~/.claude/skills/formato-curso-v5), so future courses get them too.
EOF
)"
git push origin main
```

- [ ] **Step 6: Report to the user**

State plainly: (a) the skill commit SHA on `formato-curso-inema`'s `v5-enhancements` branch (from Part A Task 9) and that it's pushed; (b) the course commit SHA on `ssh-basico`'s `main` and confirmation of `git push`; (c) the manual browser check results from Step 4; (d) that deploy to GitHub Pages is automatic (per project convention — do not poll or check Vercel/Pages status); (e) that `v5-enhancements` still needs a merge to `main` in `formato-curso-inema` to take effect for future courses — ask the user whether to merge now or leave it open for further review.

---

## Self-Review Notes (already applied above)

- **Spec coverage:** Every item from `docs/superpowers/specs/2026-07-07-v5-enhancements-design.md` has a task: `.termdemo` (Tasks 1, 4, 7, 13–18), refazer exercícios (Tasks 3, 5, 6, 7), `.fecho-wrap` (Tasks 2, 7, 12), landing header (Tasks 8, 19). Final verification (Task 20) re-checks every counter from the original publish pass so the retrofit is proven not to have regressed anything.
- **Placeholder scan:** The only literal placeholder text (`Nome do curso`, `comando --certo`, etc.) lives exclusively in the skill's own **authoring templates** (`curso-template.html`, `landing-template.html`), which are placeholders-by-design for future course authors — every course-specific file (`curso.html`, `landing.html`) has real, specific content.
- **Type/name consistency:** `resetPractice(k, scroll)` (Task 5) is called identically in Task 5 (per-aula button, `scroll=true`) and Task 6 (panel reset button, `scroll=false`) — same name, same two-argument signature, both places. `practiceRefreshers[k]` is written once (Task 5) and read once (inside `resetPractice`, also Task 5) — no other task touches it directly. `renderExercicios()` is defined once (Task 6) and called from two places also defined in Task 6 (`#exbtn` listener) — consistent.
