# Slide Hub Viewer Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Add a single hub HTML page that lets presenters choose one of the three existing seminar decks from a left navigation panel while viewing the selected deck on the right.

**Architecture:** Create `slides/index.html` as a standalone shell around the existing slide decks. The hub owns session selection and iframe layout only; each embedded deck keeps its current slide engine and controls.

**Tech Stack:** HTML, CSS, vanilla JavaScript, local iframe targets, Browser verification

---

## File Structure

- Create `slides/index.html`: Combined session navigation and iframe viewer for the three existing HTML decks.
- Leave existing deck files unchanged: `slides/junit-testcode-and-jacoco.html`, `slides/testable-code-solid-legacy-refactoring.html`, and `slides/design-patterns-for-changeable-code.html`.

### Task 1: Build the combined slide hub

**Files:**
- Create: `slides/index.html`

- [ ] **Step 1: Write the hub HTML shell**

Create `slides/index.html` with:

```html
<!DOCTYPE html>
<html lang="ko">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>Code Quality Seminar Slides</title>
</head>
<body>
  <main class="hub">
    <aside class="sidebar">
      <p class="eyebrow">Code Quality Seminar</p>
      <h1>세션 목차</h1>
      <nav aria-label="Seminar sessions">
        <button class="session active" data-src="junit-testcode-and-jacoco.html" data-title="JUnit Test Code와 JaCoCo">01 JUnit Test Code와 JaCoCo</button>
        <button class="session" data-src="testable-code-solid-legacy-refactoring.html" data-title="테스트하기 쉬운 코드: SOLID와 레거시 리팩토링">02 테스트하기 쉬운 코드</button>
        <button class="session" data-src="design-patterns-for-changeable-code.html" data-title="디자인 패턴: 변경에 강한 코드를 만드는 반복 해법">03 디자인 패턴</button>
      </nav>
    </aside>
    <section class="viewer" aria-live="polite">
      <div class="viewer-bar" id="viewer-title">JUnit Test Code와 JaCoCo</div>
      <iframe
        id="slide-frame"
        title="JUnit Test Code와 JaCoCo"
        src="junit-testcode-and-jacoco.html"></iframe>
    </section>
  </main>
</body>
</html>
```

- [ ] **Step 2: Add the responsive corporate layout**

Add inline CSS that:

- Gives the page a restrained navy/gold corporate palette
- Uses a left desktop sidebar and a right iframe viewer
- Highlights the selected `.session.active` button
- Keeps iframe space dominant
- Switches the sidebar into a compact top section below `900px`

Use this structure:

```css
:root {
  --navy: #1e3a5f;
  --gold: #c8a45c;
  --surface: #f4f5f7;
  --line: #dfe4eb;
  --ink: #333333;
}
.hub { display: grid; grid-template-columns: minmax(260px, 340px) minmax(0, 1fr); min-height: 100vh; }
.viewer { min-width: 0; display: grid; grid-template-rows: auto minmax(0, 1fr); padding: 20px; }
iframe { width: 100%; height: 100%; border: 1px solid var(--line); background: #fff; }
@media (max-width: 900px) {
  .hub { grid-template-columns: 1fr; grid-template-rows: auto minmax(0, 1fr); }
  nav { display: grid; grid-template-columns: repeat(3, minmax(0, 1fr)); }
}
```

- [ ] **Step 3: Add session switching**

Add inline JavaScript that:

- Selects all `.session` buttons
- Updates active button state
- Updates iframe `src`, `title`, and viewer title
- Keeps default load on the JUnit deck

Use:

```javascript
const frame = document.getElementById('slide-frame');
const viewerTitle = document.getElementById('viewer-title');
const sessions = [...document.querySelectorAll('.session')];

function selectSession(button) {
  sessions.forEach((session) => session.classList.toggle('active', session === button));
  frame.src = button.dataset.src;
  frame.title = button.dataset.title;
  viewerTitle.textContent = button.dataset.title;
}

sessions.forEach((button) => {
  button.addEventListener('click', () => selectSession(button));
});
```

- [ ] **Step 4: Run static verification**

Run:

```powershell
git diff --check -- slides/index.html
Select-String -Path 'slides\index.html' -SimpleMatch 'junit-testcode-and-jacoco.html','testable-code-solid-legacy-refactoring.html','design-patterns-for-changeable-code.html'
```

Expected:

- `git diff --check` exits `0`
- All three deck filenames are present in `slides/index.html`

- [ ] **Step 5: Verify in the browser**

Open `slides/index.html` through a local preview URL and verify:

- Default iframe source points at `junit-testcode-and-jacoco.html`
- Clicking the SOLID and design-pattern session buttons swaps the iframe source
- Active button state moves with the selected session
- The embedded deck still exposes its own previous/next controls
- A desktop screenshot shows left navigation and right slide viewer without overlap

- [ ] **Step 6: Check repository scope**

Run:

```powershell
git status --short --branch
git diff --name-only
```

Expected:

- New implementation change is `slides/index.html`
- Existing deck HTML files are not modified by this task
- Unrelated untracked `refactoring-guru-catalog.md` remains untouched
