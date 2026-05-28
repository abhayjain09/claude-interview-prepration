# Interview Prep App Builder — Project Context

## What This Project Does
Builds self-contained interactive HTML study apps for technical interview and coding prep.
Output: one `.html` file with sidebar nav, reveal-on-click answers, Study / Mock / Flashcard modes,
and progress tracking. No server or dependencies needed.

---

## My Profile (Abhay Lunkad)
- **Current role:** Lead Platform Engineer, S&P Global
- **Target role:** Senior MLOps / AI Platform Engineer at AHEAD
- **Stack:** AWS (ECS/EKS/Bedrock/Lambda/OpenSearch), Terraform, GitHub Actions, ArgoCD,
  Python, Docker, LangSmith, Langfuse, OpenTelemetry, CloudWatch, Grafana/Prometheus
- **Key projects to reference:**
  - AgentCore cost-optimization framework (multi-agent, cross-account EC2/RDS rightsizing, Bedrock + Lambda tools, persistent DynamoDB memory, Guardrails)
  - RAG pipeline on Bedrock (Claude + Titan Embeddings + OpenSearch Serverless, 50K+ Terraform docs, sub-2s p95)
  - CI/CD consolidation (15+ pipelines → GitHub Actions standard, 40% error reduction)
  - Amazon Connect outreach automation (1,200+ hrs automated)
  - Security Hub posture improvement (+30%)

---

## How to Build a New App

**Step 1 — I will tell you:**
> "Create a prep app for [TOPIC]. Topics: [list]. Use [accent color] accent."

**Step 2 — You build using this exact pattern:**

### Data Schema
```javascript
const TOPICS = [
  {
    id: "topic-id",
    name: "Display Name",
    ix: 1,
    group: "Group Name",
    concept: `<p>HTML explanation...</p><ul><li>...</li></ul>`,
    questions: [
      {
        level: "beginner|intermediate|advanced|scenario|architecture|leadership|easy|medium|hard|platform|system",
        q: "Question text",
        code: `<pre>...</pre>`,   // coding topics only
        a: "Answer HTML",
        top1: "What top 1% says that good candidates don't",
        pitfall: "Most common mistake",  // optional
        followups: ["Follow-up 1", "Follow-up 2"]
      }
    ],
    mistakes: ["..."],
    bestPractices: ["..."],   // interview topics
    tradeoffs: ["..."],       // interview topics
    patterns: ["..."],        // coding topics
    complexity: [{op:"", avg:"", worst:""}]  // coding topics, optional
  }
];  // END TOPICS
```

### Build Process
```bash
# Shell: use heredoc appends, never use backticks inside heredoc content
cat >> output.html << 'EOF'
...content...
EOF
```

Split into chunks:
1. HTML head + CSS + body shell
2. `const TOPICS = [` + first batch of topics
3. More topics per chunk
4. Last chunk: close TOPICS array + supplementary data + JS app logic + `</script></body></html>`

### Mandatory Validation (run after build)
```python
checks = {
    "TOPICS closed":           "]; // END TOPICS" in content,
    "qbody hidden":            ".qbody{display:none" in content,
    "q.open shows qbody":      ".q.open .qbody{display:block}" in content,
    "toggleQ uses closest":    "el.closest('.q')" in content,
    "reveal pattern":          "reveal-prompt" in content,
    "no mobnav getElementById":"getElementById('mobnav')" not in content,
    "storage guarded":         "if(window.storage)" in content,
    "HTML closes":             content.strip().endswith("</html>"),
}
```

---

## Critical Rules (bugs that crashed previous builds)

### 1. toggleQ — must use `closest('.q')`, never `nextElementSibling`
```javascript
// ✅ CORRECT
function toggleQ(el) {
  const card = el.closest('.q');
  card.classList.toggle('open');
}
// ❌ WRONG — toggles on qbody, not on .q parent → CSS never matches
function toggleQ(el) {
  const body = el.nextElementSibling;
  body.classList.toggle('open');
}
```

### 2. Never `getElementById('mobnav')` in JS
```html
<!-- ✅ CORRECT — inline onclick -->
<button class="mbtn mobnav" onclick="document.getElementById('side').classList.toggle('open')">☰</button>
```
```javascript
// ❌ WRONG — mobnav is a CLASS not an ID, returns null, crashes the whole app
document.getElementById('mobnav').onclick = function() { ... }
```

### 3. THREE CSS rules required for question expand (never omit any)
```css
.qbody { display: none; }            /* hidden by default */
.q.open { border-color: var(--line2); }
.q.open .qbody { display: block; }  /* ← most commonly missing, breaks everything */
```

### 4. window.storage must be guarded
```javascript
// ✅ CORRECT
if (window.storage) {
  const r = await window.storage.get('key');
} else {
  const ls = localStorage.getItem('key');
}
// ❌ WRONG — window.storage undefined in some environments → crash
await window.storage.get('key');
```

### 5. buildNav uses divs with class 'navitem', not anchor tags
CSS must define `.navitem` (not `.nav a`) because JS creates `div.navitem` elements:
```css
.navitem { display: block; padding: 7px 20px; cursor: pointer; ... }
.navitem.active { background: ...; color: var(--accent); }
```

---

## CSS Design System

```css
:root {
  --bg: #0a0c10;      --bg2: #0f1218;     --panel: #151a22;
  --line: #1e2535;    --line2: #2a3347;   --ink: #e2e8f0;
  --muted: #7a8a9e;   --faint: #3d4f68;
  /* Accent colors — pick one as primary per domain */
  --amber: #e8b454;   /* interview/devops */
  --green: #3dffa0;   /* coding/python */
  --cyan: #38d9f5;    /* AI/ML */
  --red: #ff5c7a;     /* security */
  --purple: #b07cff;  /* highlight */
}
```

**Fonts by domain:**
- Interview apps: `Fraunces` + `IBM Plex Sans` + `JetBrains Mono`
- Coding apps: `Syne` + `DM Sans` + `Space Mono`

---

## Content Quality Standards

**Per topic:** 5–7 questions minimum, covering easy/medium/hard + scenario + architecture
**`top1` field:** This is the premium differentiator — what a top 1% answer includes that a good answer doesn't. Be specific: name tools, numbers, edge cases, when-NOT-to-use, tradeoffs
**Tailor to my profile:** Reference my actual projects (AgentCore, RAG pipeline, CI/CD consolidation), tech stack, and S&P Global / financial services context
**mistakes / bestPractices / tradeoffs:** 5–7 each, specific not generic

---

## Modes to Include

| Mode | Description |
|---|---|
| **Study** | Sidebar nav → topic → question list → reveal-on-click answer → self-rate |
| **Mock Interview** | 50 random questions from all topics, reveal → rate → next |
| **Flash Cards** | 40–50 rapid-fire Q&A cards, shuffleable |

---

## Supplementary Sections (interview prep)
- `STAR_ANSWERS` — polished STAR stories from my resume
- `RAPID_FIRE` — 40–50 short flashcard Q&As
- `DEBUG_SCENARIOS` — hands-on troubleshooting walkthroughs
- `OUTAGES` — outage simulations with step-by-step response

---

## Example Request Format

```
Create a prep app for [TOPIC].
Topics to cover: [list of 8-24 topics]
Accent color: [amber|green|cyan|red]
File name: [name]_prep.html
```

Or simply:
```
Same pattern — create a prep app for Go/Kubernetes/System Design/etc.
```
