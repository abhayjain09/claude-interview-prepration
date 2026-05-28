---
name: interview-prep-app-builder
description: Build a complete single-file interactive HTML interview/coding prep study app for a specific domain or role. Use this skill whenever the user asks to create an interview prep app, coding round prep, study guide app, or learning tool for technical topics. Also triggers when they say "create a prep app for X", "build me a study tool for Y interview", "make a coding prep file for Z", or "create context/skill for topic X using the same pattern". The skill produces a self-contained HTML file with sidebar navigation, reveal-on-click answers, 3 study modes (Study/Mock/Flashcards), progress tracking, and senior-level content tailored to the user's profile.
---

# Interview Prep App Builder

Builds a self-contained interactive HTML study app for technical interview or coding prep. The output is one `.html` file the user can open locally or share — no server, no dependencies except Google Fonts.

## When to use this skill

- User asks to create a prep app, study tool, or learning guide for a technical domain
- User says "same pattern as before" or "new topic using the same approach"
- User provides a topic list and wants an interactive app to study it
- User wants a coding round prep file for a specific language/framework

---

## Output Architecture

Single HTML file with three embedded sections:
1. **`<style>`** — Full CSS design system (dark console theme, CSS variables, all component styles)
2. **`<script>` data** — `const TOPICS = [...]` array + supplementary data arrays
3. **`<script>` app logic** — Nav builder, renderers, state management, mode switching

### Build Method (for large files)
Use `cat >> file.html << 'EOF'` heredoc appends in bash. Never use backticks inside heredoc content — use `'` quotes or HTML entities instead. Split into logical chunks:
- Chunk 1: HTML head + full CSS + body shell
- Chunks 2–N: One or more TOPICS per append
- Final chunk: Supplementary data + full JS app logic + closing tags

Always validate with Python after building:
```python
checks = {
    "TOPICS array closed": "]; // END TOPICS" in content,
    "q.open shows qbody": ".q.open .qbody{display:block}" in content,
    "reveal pattern": "reveal-prompt" in content,
    "toggleQ uses closest": "el.closest('.q')" in content,
    "no mobnav getElementById": "getElementById('mobnav')" not in content,
    "storage guarded": "if(window.storage)" in content,
    "HTML closes": content.strip().endswith("</html>"),
}
```

---

## Data Schema

### TOPICS Array
Each topic object:
```javascript
{
  id: "topic-id",           // kebab-case, unique
  name: "Display Name",     // shown in sidebar and crumb
  ix: 1,                    // display number
  group: "Group Name",      // sidebar group header
  concept: `<p>...</p><ul>...</ul>`,  // HTML string, 200-400 words, explains the topic deeply
  questions: [
    {
      level: "beginner"|"intermediate"|"advanced"|"scenario"|"architecture"|"leadership"|
             "easy"|"medium"|"hard"|"platform"|"system",
      q: "Question text",
      code: `<pre>...</pre>`,   // optional — for coding topics
      a: "Answer text (HTML ok)",
      top1: "What separates a top 1% answer from a good answer",
      pitfall: "Most common mistake (optional)",
      followups: ["Follow-up question 1", "Follow-up question 2"]
    }
  ],
  mistakes: ["Common mistake 1", "Common mistake 2"],
  // For interview topics:
  bestPractices: ["Best practice 1"],
  tradeoffs: ["Tradeoff 1"],
  // For coding topics:
  patterns: ["Pattern 1"],
  complexity: [{op: "operation", avg: "O(n)", worst: "O(n²)"}]  // optional
}
```

### Supplementary Data Arrays
Include as appropriate for the domain:

**Interview prep:**
```javascript
const STAR_ANSWERS = [{id, title, s, t, a, r}];  // STAR resume stories
const RAPID_FIRE = [{q, a}];                       // 40-50 short flashcard Q&As
const DEBUG_SCENARIOS = [{title, symptoms, diagnosis, fix}];
const OUTAGES = [{title, severity, scenario, steps:[], lesson}];
```

**Coding prep:**
```javascript
const FLASH = [{q, a}];  // 40+ concept flashcards
```

---

## CSS Design System

The dark console theme uses these CSS variables — keep consistent across all apps:

```css
:root {
  --bg: #0a0c10;           /* page background */
  --bg2: #0f1218;          /* sidebar background */
  --panel: #151a22;        /* card/panel background */
  --panel2: #1c2330;       /* nested panel */
  --line: #1e2535;         /* border */
  --line2: #2a3347;        /* active border */
  --ink: #e2e8f0;          /* primary text */
  --muted: #7a8a9e;        /* secondary text */
  --faint: #3d4f68;        /* disabled/label text */
  --amber: #e8b454;        /* primary accent (interview prep) */
  --green: #3dffa0;        /* primary accent (coding prep) */
  --cyan: #38d9f5;         /* secondary accent */
  --red: #ff5c7a;          /* danger/error */
  --purple: #b07cff;       /* highlight */
}
```

**Accent color by domain:**
- Interview/DevOps prep: amber (#e8b454) as primary
- Coding/algorithm prep: green (#3dffa0) as primary
- Security prep: red (#ff5c7a) as primary
- AI/ML prep: cyan (#38d9f5) as primary

**Typography stack:**
- Interview apps: `Fraunces` (display) + `IBM Plex Sans` (body) + `JetBrains Mono` (code)
- Coding apps: `Syne` (display) + `DM Sans` (body) + `Space Mono` (code)

---

## Critical CSS Rules (never omit)

```css
/* Question expand/collapse — ALL THREE rules required */
.qbody { display: none; }           /* hidden by default */
.q.open .qbody { display: block; }  /* shown when open — THIS IS THE KEY RULE */
.q.open { border-color: var(--line2); }

/* Reveal pattern */
.reveal-prompt { padding: 14px 0 6px; }
.reveal-btn { /* styled button */ cursor: pointer; }

/* Sidebar nav — JS creates these class names */
.navgrp { margin: 8px 0 2px; }
.navgrplbl { display: block; /* group header */ }
.navitem { display: block; cursor: pointer; /* topic links */ }
.navitem.active { /* highlighted state */ }
```

---

## Critical JS Patterns (never deviate)

### toggleQ — must use closest('.q'), not nextElementSibling
```javascript
function toggleQ(el) {
  const card = el.closest('.q');          // ← CORRECT: toggles open on the .q parent
  const icon = el.querySelector('.qtoggle');
  const open = card.classList.toggle('open');
  icon.textContent = open ? '▾' : '▸';
}
```

### revealAnswer — reveal-prompt → answer-section pattern
```javascript
function revealAnswer(btn) {
  const wrap = btn.closest('.qbody');
  wrap.querySelector('.reveal-prompt').style.display = 'none';
  wrap.querySelector('.answer-section').style.display = 'block';
}
```

### Storage — always guard window.storage
```javascript
async function loadProgress() {
  try {
    if (window.storage) {
      const r = await window.storage.get('key');
      if (r && r.value) progress = JSON.parse(r.value);
    } else {
      const ls = localStorage.getItem('key');
      if (ls) progress = JSON.parse(ls);
    }
  } catch(e) { progress = {}; }
}
```

### Mobile nav — use inline onclick, never getElementById('mobnav')
```html
<button class="mbtn mobnav" onclick="document.getElementById('side').classList.toggle('open')">☰</button>
```
**Never do:** `document.getElementById('mobnav').onclick = ...` — mobnav is a CLASS not an ID, this crashes the app.

### buildNav — creates .navgrp / .navitem divs (not anchor tags)
```javascript
function buildNav() {
  const groups = {};
  TOPICS.forEach(t => { if (!groups[t.group]) groups[t.group] = []; groups[t.group].push(t); });
  const nav = document.getElementById('nav');
  nav.innerHTML = '';
  Object.entries(groups).forEach(([g, ts]) => {
    const grpEl = document.createElement('div');
    grpEl.className = 'navgrp';
    grpEl.innerHTML = '<span class="navgrplbl">' + g + '</span>';
    ts.forEach(t => {
      const li = document.createElement('div');
      li.className = 'navitem';
      li.dataset.id = t.id;
      li.textContent = t.ix + '. ' + t.name;
      li.onclick = () => showTopic(t.id);
      grpEl.appendChild(li);
    });
    nav.appendChild(grpEl);
  });
}
```

---

## Question HTML Template

```javascript
const qHTML = t.questions.map((q, i) => `
  <div class="q qlvl-${q.level}" data-lvl="${q.level}">
    <div class="qhead" onclick="toggleQ(this)">
      <span class="lvl ${q.level}">${levelLabel(q.level)}</span>
      <span class="qtext">${q.q}</span>
      <span class="qtoggle">▸</span>
    </div>
    <div class="qbody">
      <div class="reveal-prompt">
        <p>Think through your answer, then reveal.</p>
        <button class="reveal-btn" onclick="revealAnswer(this)">🔍 Reveal Answer</button>
      </div>
      <div class="answer-section" style="display:none">
        ${q.code || ''}
        <div class="ans"><span class="albl">Answer</span>${q.a}</div>
        <div class="top1"><strong>🏆 Top 1%:</strong> ${q.top1}</div>
        ${q.pitfall ? '<div class="pitfalls">⚠️ ' + q.pitfall + '</div>' : ''}
        ${q.followups?.length ? '<div class="followups"><ul>' + q.followups.map(f => '<li>' + f + '</li>').join('') + '</ul></div>' : ''}
        <div class="selfrate">
          <button class="rate sel-good" onclick="rateQuestion('${id}','good')">✅ Got it</button>
          <button class="rate sel-mid" onclick="rateQuestion('${id}','mid')">😐 Partial</button>
          <button class="rate sel-bad" onclick="rateQuestion('${id}','bad')">❌ Missed</button>
        </div>
      </div>
    </div>
  </div>`).join('');
```

---

## Content Quality Standards

### Questions per topic: 5–7 minimum
- Cover beginner → advanced → scenario/architecture progression
- Always include at least one scenario/troubleshooting question
- Always include at least one architecture/design question for senior roles
- For coding topics: always include a real working code solution

### The "top1" field is the most important
This is what makes the content premium. It should answer: "What does a top 1% candidate say that a good candidate doesn't?"
- Names specific tools, numbers, edge cases
- Shows judgment ("when NOT to use this")
- Connects to the user's actual resume/experience
- Reveals trade-offs the average candidate misses

### Tailor to the user's profile
Always connect questions and answers to the user's actual work:
- Reference their specific projects, companies, tech stack
- Use their numbers (e.g., "your sub-2s p95 RAG pipeline")
- Frame examples in their industry (financial services = regulatory, audit, cost governance)

### mistakes / bestPractices / tradeoffs: 5–7 each
Should be specific, not generic. Bad: "Don't use X." Good: "Using X without Y causes Z in production."

---

## Mock Interview Mode

```javascript
// Pull from TOPICS questions for mock pool
mockPool = TOPICS.flatMap(t => t.questions.map(q => ({topic: t.name, ...q})))
  .sort(() => Math.random() - 0.5)
  .slice(0, 50);  // 50 questions for a full mock
```

---

## File Naming Convention

| Domain | Filename |
|---|---|
| Role + company interview prep | `{company}_{role}_interview_prep.html` |
| Language coding round | `{language}_coding_prep.html` |
| Topic-specific prep | `{topic}_prep.html` |

---

## Quick-Start Template for a New Topic

When the user provides a new domain, use this sequence:

1. **Ask for (or infer from context):**
   - Target role/company
   - User's background/resume highlights
   - 8–24 topic areas to cover
   - Domain accent color preference (or choose based on domain)

2. **Build order:**
   ```
   Chunk 1: HTML head + full CSS (adapt accent color) + body HTML shell
   Chunk 2: const TOPICS = [ + first 4-6 topics
   Chunk 3: Next 4-6 topics
   ...
   Last chunk: ]; // END TOPICS + supplementary data + full JS logic + </script></body></html>
   ```

3. **Validate** with the Python checks listed above

4. **Copy to outputs** and call `present_files`

---

## Common Failure Modes to Avoid

| Bug | Symptom | Fix |
|---|---|---|
| Missing `.q.open .qbody{display:block}` CSS | Clicking question does nothing | Add the rule — it's always needed |
| `getElementById('mobnav')` in JS | Crashes entire app on load | Use inline onclick on the button instead |
| `body.classList.toggle('open')` in toggleQ | Questions never expand | Use `el.closest('.q').classList.toggle('open')` |
| Duplicate `.qbody{display:block}` rule | Answers always visible | Remove the duplicate that overrides the `display:none` |
| window.storage without guard | Crashes in some environments | Always wrap with `if(window.storage)` |
| Backtick in heredoc content | Heredoc terminates early | Use `'` or HTML entity `&#96;` inside heredoc |
| Mutable default in `buildNav` | Nav rebuilds wrong | Always set `nav.innerHTML=''` before building |

