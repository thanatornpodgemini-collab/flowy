# Codex Handoff — PMM Agentic Workflow Designer

## Implementation log

**2026-05-03 — All 12 fixes applied by Codex and synced.**

| # | Fix | Status |
|---|-----|--------|
| 1 | `r_array[0].y` in `rearrangeMe` | ✅ Done |
| 2 | Bottom-edge `scrollTop` typo | ✅ Done |
| 3 | Empty canvas clears localStorage | ✅ Done |
| 4 | XSS via innerHTML — replaced with `createElement` + `textContent` | ✅ Done |
| 5 | Analytics label → ID matching | ✅ Done |
| 6 | `innerHTML +=` → `insertAdjacentHTML` in `drawArrow` | ✅ Done |
| 7 | Silent catch blocks now log errors | ✅ Done |
| 8 | Form input debounced at 150ms | ✅ Done |
| 9 | Int/string type mismatch in `snap` rearrange | ✅ Done |
| 10 | `flowy.destroy()` cleanup method added | ✅ Done |
| 11 | Hot-path `.filter()` calls cached per loop | ✅ Done |
| 12 | Dead code removed (`addEventListenerMulti`, `removeEventListenerMulti`, `maxheight`) | ✅ Done |

Both files passed `node --check` syntax validation. Fixed files synced to `/Users/cashclaw/Documents/New project 5`.

**2026-05-03 — Starter workflow expanded to 14 steps by Codex.**

Replaced the 7-step placeholder in `demo/main.js` (`build-starter` handler) with a full end-to-end PMM launch lifecycle:

| # | Step | Owner | Bucket | Risk |
|---|------|-------|--------|------|
| 1 | Market & Competitive Research | PMM Lead | assist | medium |
| 2 | Customer & ICP Discovery | PMM + UX Researcher | human | medium |
| 3 | Positioning Workshop | PMM Lead + CPO | human | high |
| 4 | Messaging Architecture | PMM Lead | human | high |
| 5 | Message Testing & Validation | PMM + Demand Gen | assist | medium |
| 6 | Go-to-Market Strategy | PMM Lead + VP Marketing | human | high |
| 7 | Content Brief Creation | PMM | assist | low |
| 8 | Sales Enablement Kit | PMM + Sales Ops | assist | medium |
| 9 | Launch Asset Production | Content Team + PMM | assist | medium |
| 10 | Analyst & Press Briefings | PMM Lead + PR | human | high |
| 11 | Internal Launch Readiness | PMM + RevOps | assist | low |
| 12 | Launch Execution | PMM + Demand Gen + PR | assist | high |
| 13 | Localization & Regional Adaptation | PMM + Regional Leads | agent | medium |
| 14 | Post-Launch Performance Review | PMM Lead + Analytics | assist | low |

Click **"Build Starter Flow"** in the app to load this workflow for editing.

**2026-05-03 — Expanded to full branching PMM workflow tree (21 nodes) by Codex.**

Replaced the linear 14-step chain with a trunk + 4 operational branches. `buildStarterFlowImport` now uses a recursive subtree-width centering layout (root x=600 y=160, +220px depth, 320px sibling gap).

**Trunk — Launch Lifecycle (IDs 0–7, linear)**
Market Research → ICP Discovery → Positioning Workshop → Messaging Architecture → GTM Strategy → Launch Assets → Launch Execution → Post-Launch Review

**Branch A — Ongoing Competitive Intel (IDs 8–10, off ID 0)**
Weekly Signal Scan → Battlecard Refresh → Sales Alert Broadcast

**Branch B — Content Calendar Loop (IDs 11–14, off ID 5)**
Monthly Content Planning → SEO & Keyword Review → Content Production Briefing → Multi-Channel Publishing

**Branch C — Win/Loss & Pipeline Review (IDs 15–17, off ID 7)**
Weekly Win/Loss Pull → Messaging Resonance Analysis → Quarterly Positioning Refresh

**Branch D — Sales Enablement Ops (IDs 18–20, off ID 4)**
Sales Feedback Collection → Enablement Asset Update → Sales Training Session

`node --check` passes. Synced to `/Users/cashclaw/Documents/New project 5`.

---

## Project overview

A drag-and-drop workflow designer for Product Marketing Managers, built on top of the open-source **Flowy.js** library. Users compose workflow graphs from template blocks, assign metadata (owner, tool, risk, automation %), and get live analytics on workload and automation potential.

**Key files:**
- `engine/flowy.js` — core drag-and-drop engine (third-party, lightly modified)
- `demo/main.js` — application logic (block hydration, analytics, persistence)
- `demo/index.html` — single-page UI
- `demo/styles.css` — layout and component styles

---

## ~~Open issue — branching tree not rendering~~ — RESOLVED (2026-05-03)

**Root cause:** `flowy.import()` in `engine/flowy.js` calls `rearrangeMe()` and `checkOffset()` after setting `canvas_div.innerHTML`. `rearrangeMe()` recalculates and overwrites every block's `left`/`top` using flowy's own linear stacking algorithm — completely discarding the custom tree positions set in `buildStarterFlowImport`. Branching nodes all get stacked in a single vertical column.

**Secondary cause:** `restore()` on DOMContentLoaded loads the last-saved flow from localStorage, so clicking "Build Starter Flow" while stale data is cached appears to do nothing until localStorage is cleared.

### Fix 1 — Skip rearrangeMe on import (`engine/flowy.js`)

Add a `skipRearrange` parameter to `flowy.import` so callers that supply pre-positioned HTML can opt out:

```js
// engine/flowy.js — inside flowy.load()
flowy.import = function(output, skipRearrange) {
    canvas_div.innerHTML = output.html;
    for (var a = 0; a < output.blockarr.length; a++) {
        blocks.push({
            childwidth: parseFloat(output.blockarr[a].childwidth),
            parent: parseFloat(output.blockarr[a].parent),
            id: parseFloat(output.blockarr[a].id),
            x: parseFloat(output.blockarr[a].x),
            y: parseFloat(output.blockarr[a].y),
            width: parseFloat(output.blockarr[a].width),
            height: parseFloat(output.blockarr[a].height)
        });
    }
    if (!skipRearrange && blocks.length > 1) {
        rearrangeMe();
        checkOffset();
    }
}
```

### Fix 2 — Pass `skipRearrange=true` from the build-starter handler (`demo/main.js`)

```js
// demo/main.js — inside build-starter click handler
flowy.import(buildStarterFlowImport(blocks), true);   // ← add second arg
```

### Fix 3 — Also pass `skipRearrange=true` from restore() to preserve saved layouts

```js
// demo/main.js — inside restore()
flowy.import(JSON.parse(raw), true);
```

### Fix 4 — Clear localStorage before loading starter flow

In the build-starter click handler, clear the saved key before importing so restore() on the next page load shows the new tree, not the old linear flow:

```js
document.getElementById("build-starter").addEventListener("click", () => {
    flowy.deleteBlocks();
    selectedBlock = null;
    localStorage.removeItem(STORAGE_KEY);   // ← add this line
    const blocks = [ ... ];
    flowy.import(buildStarterFlowImport(blocks), true);
    ...
});
```

**Files to change:** `engine/flowy.js` (Fix 1) and `demo/main.js` (Fixes 2–4).  
**Do not change** the `buildStarterFlowImport` layout logic or the blocks array — those are correct.

---

## Bugs to fix

### 1. `rearrangeMe` uses array object instead of array element — blocks misplace on rearrange
**File:** `engine/flowy.js` lines 596–607  
`blocks.filter()` returns an array `[]`, but the code reads and writes `.y` directly on that array instead of `[0].y`. Block positions become `NaN` after any rearrange.

```js
// Current (broken)
const r_array = blocks.filter(id => id.id == result[z]);
r_block.style.top = r_array.y + paddingy + canvas_div.getBoundingClientRect().top - absy + "px";
r_array.y = r_array.y + paddingy;
r_block.style.left = r_array[0].x - ...  // inconsistent — some lines already use [0]

// Fix: use r_array[0] consistently
r_block.style.top = r_array[0].y + paddingy + canvas_div.getBoundingClientRect().top - absy + "px";
r_array[0].y = r_array[0].y + paddingy;
```

### 2. Bottom-edge auto-scroll uses wrong axis
**File:** `engine/flowy.js` line 514  
When the cursor hits the bottom edge of the canvas, the code scrolls `scrollLeft` instead of `scrollTop`.

```js
// Current (broken)
} else if (mouse_y < canvas_div.getBoundingClientRect().top + 10 && mouse_y > canvas_div.getBoundingClientRect().top - 10) {
    canvas_div.scrollLeft -= 10;

// Fix
    canvas_div.scrollTop -= 10;
```

### 3. Rearrange snap has int/string type mismatch
**File:** `engine/flowy.js` line 329  
All other `blockstemp.filter` calls use `parseInt()` on the `.value`, but this one does not — the `==` coercion may silently fail to find the block in strict environments or after refactoring.

```js
// Current
blockstemp.filter(a => a.id == drag.querySelector(".blockid").value)[0].parent = blocko[i];

// Fix
blockstemp.filter(a => a.id === parseInt(drag.querySelector(".blockid").value))[0].parent = blocko[i];
```

### 4. Analytics auto-correct matches blocks by label instead of ID
**File:** `demo/main.js` lines 157–160  
When a high-risk block is missing an approval gate, the fix iterates all canvas blocks and matches by `label` string. Two blocks with identical labels both get mutated.

```js
// Current (fragile)
document.querySelectorAll("#canvas .block").forEach((node) => {
  if (getField(node, "label") === b.dataMap.label) {
    setField(node, "approval", "true");
  }
});

// Fix: match by block id
document.querySelectorAll("#canvas .block").forEach((node) => {
  if (parseInt(getField(node, "blockid")) === b.id) {
    setField(node, "approval", "true");
  }
});
```

### 5. Clearing the canvas doesn't clear localStorage
**File:** `demo/main.js` lines 271–275  
`persist()` bails early when `flowy.output()` returns falsy (empty canvas), leaving the previous flow in localStorage. On next page load the cleared canvas is restored.

```js
// Current
function persist() {
  const out = flowy.output();
  if (!out) return;
  localStorage.setItem(STORAGE_KEY, JSON.stringify(out));
}

// Fix
function persist() {
  const out = flowy.output();
  if (!out) {
    localStorage.removeItem(STORAGE_KEY);
    return;
  }
  localStorage.setItem(STORAGE_KEY, JSON.stringify(out));
}
```

---

## Security fixes

### 6. XSS via innerHTML string interpolation
**Files:** `demo/main.js` lines 87–93, 207–229  
`hydrateBlock` and `buildStarterFlowImport` interpolate user-controlled values (`step.label`, `step.person`, `step.tool`, `step.output`, `step.risk`) directly into HTML strings. A crafted import JSON or typed label like `<img src=x onerror=alert(1)>` executes.

**Fix:** Render container elements with `createElement`, then set text with `textContent` rather than building HTML strings. Example for `hydrateBlock`:

```js
function hydrateBlock(block) {
  // ... setup ...
  const card = document.createElement("div");
  card.className = "canvas-card";

  const h4 = document.createElement("h4");
  h4.className = "node-label";
  h4.textContent = title;          // safe — no HTML parsing

  const pOwner = document.createElement("p");
  pOwner.innerHTML = "<strong>Owner:</strong> ";
  const ownerSpan = document.createElement("span");
  ownerSpan.className = "node-person";
  ownerSpan.textContent = defaults.person;
  pOwner.appendChild(ownerSpan);

  // ... repeat for other fields ...
  card.append(h4, pOwner, ...);
  block.appendChild(card);
}
```

Apply the same pattern to `buildStarterFlowImport` or sanitize all string fields with a helper before interpolation.

### 7. `innerHTML +=` in `drawArrow` re-serializes the entire canvas
**File:** `engine/flowy.js` lines 277–281  
`canvas_div.innerHTML += '...'` serializes all existing DOM, concatenates a string, then re-parses everything. This destroys and recreates all existing nodes (event listeners lost, slow) and re-parses any unsafe content already in the DOM.

**Fix:** Use `insertAdjacentHTML('beforeend', ...)` or create elements via `createElement`. This is the minimal-change fix:

```js
// Replace: canvas_div.innerHTML += '<div class="arrowblock">...'
canvas_div.insertAdjacentHTML('beforeend', '<div class="arrowblock">...');
```

---

## Reliability improvements

### 8. Silent error swallowing hides broken state
**File:** `demo/main.js` lines 286–287, 362–363  
Both `restore()` and the JSON import handler have empty `catch` blocks. Corrupt localStorage or an invalid JSON file fails invisibly.

```js
// Fix: at minimum log, ideally show user feedback
} catch (err) {
  console.error("Failed to restore workflow:", err);
  // Optional: show a non-blocking error banner in the UI
}
```

### 9. No cleanup/destroy for event listeners
**File:** `engine/flowy.js` lines 617–628  
All listeners are attached to `document` with no removal path. Calling `flowy()` a second time or navigating away in a SPA context accumulates duplicate listeners.

**Fix:** Return a `destroy` function from `flowy.load()` that removes all registered listeners:

```js
function cleanup() {
  document.removeEventListener("mousedown", flowy.beginDrag);
  document.removeEventListener("mousedown", touchblock);
  // ... etc
}
flowy.destroy = cleanup;
```

---

## Performance improvements

### 10. Debounce the properties form input handler
**File:** `demo/main.js` line 290  
Every keystroke calls `refreshAnalytics()` which calls `flowy.output()` and iterates all DOM blocks. Add a 150ms debounce.

```js
let analyticsTimer;
form.addEventListener("input", (e) => {
  if (!selectedBlock || !e.target.name) return;
  const value = e.target.type === "checkbox" ? String(e.target.checked) : e.target.value;
  setField(selectedBlock, e.target.name, value);
  if (e.target.name === "risk" && e.target.value === "high") {
    setField(selectedBlock, "approval", "true");
    form.elements.namedItem("approval").checked = true;
  }
  updateNodeCard(selectedBlock);
  clearTimeout(analyticsTimer);
  analyticsTimer = setTimeout(refreshAnalytics, 150);
});
```

### 11. Cache repeated `blocks.filter()` lookups
**File:** `engine/flowy.js` — multiple functions  
`snap()`, `checkOffset()`, and `rearrangeMe()` call `blocks.filter(a => a.id == id)[0]` 4–8 times per block with the same id. Cache the result once per loop iteration.

---

## Dead code to remove

- `engine/flowy.js` lines 647–659: `addEventListenerMulti` and `removeEventListenerMulti` are defined but never called anywhere.
- `engine/flowy.js` lines 301, 568: `var maxheight = 0` declared in `snap()` and `rearrangeMe()` but never read or updated.

---

## Suggested fix order

| # | Fix | Risk | Effort |
|---|-----|------|--------|
| 1 | `r_array[0].y` in rearrangeMe | High impact, low risk | Small |
| 2 | Bottom-edge scrollTop typo | High impact, trivial | Trivial |
| 3 | localStorage clear on empty canvas | Medium impact, low risk | Trivial |
| 4 | XSS via innerHTML interpolation | Security, medium effort | Medium |
| 5 | Analytics label→ID matching | Correctness, low risk | Small |
| 6 | `innerHTML +=` → `insertAdjacentHTML` | Perf + security | Small |
| 7 | Silent catch blocks | Reliability | Trivial |
| 8 | Form input debounce | Perf | Small |
| 9 | Int/string type mismatch in snap | Correctness | Trivial |
| 10 | Destroy/cleanup method | Reliability | Medium |
| 11 | Remove dead code | Cleanliness | Trivial |
