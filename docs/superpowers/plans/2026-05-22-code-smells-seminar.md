# Code Smells Seminar Implementation Plan

> **For agentic workers:** REQUIRED SUB-SKILL: Use superpowers:subagent-driven-development (recommended) or superpowers:executing-plans to implement this plan task-by-task. Steps use checkbox (`- [ ]`) syntax for tracking.

**Goal:** Validate the English Code Smells summary and create a Korean Wiki document plus a presentation-ready seminar slide deck.

**Architecture:** Keep the English source unchanged and produce two independent seminar artifacts: a Korean Markdown reading document at the repository root and a standalone HTML slide deck under `slides/`. Reuse the existing slide deck interaction model and visual tone so the new deck behaves like the current seminar decks while the Markdown remains suitable for internal Wiki distribution.

**Tech Stack:** Markdown, standalone HTML, CSS, vanilla JavaScript, Java code snippets, GitHub Pages-compatible static files

---

## File Map

- Create `code-smells-ko.md`
  - Korean Wiki-style seminar document with validated code-smell guidance and team-facing interpretation notes.
- Create `slides/code-smells.html`
  - Standalone 16:9 seminar slide deck with progress, counter, keyboard/touch navigation, fullscreen, speaker notes popup, diagrams, tables, and Java examples.
- Read `Code-Smells.md`
  - English source catalog used for validation only; do not modify it.
- Read `slides/junit-testcode-and-jacoco.html`
  - Existing deck interaction and visual reference.
- Read `slides/design-patterns-for-changeable-code.html`
  - Existing slide composition and code example reference.

### Task 1: Validate Source Coverage

**Files:**
- Read: `Code-Smells.md`
- Read: `refactoring-guru-catalog.md`
- Read: `docs/superpowers/specs/2026-05-22-code-smells-seminar-design.md`

- [ ] **Step 1: Compare the source smell groups**

  Read the source headings and confirm the five groups are present:

  ```text
  Bloaters
  Object-Orientation Abusers
  Change Preventers
  Dispensables
  Couplers
  ```

- [ ] **Step 2: Compare source smell names**

  Confirm the source contains these smell names before writing the Korean artifact:

  ```text
  Long Method
  Large Class
  Primitive Obsession
  Long Parameter List
  Data Clumps
  Switch Statements
  Temporary Field
  Refused Bequest
  Alternative Classes with Different Interfaces
  Divergent Change
  Shotgun Surgery
  Parallel Inheritance Hierarchies
  Comments
  Duplicate Code
  Lazy Class
  Data Class
  Dead Code
  Speculative Generality
  Feature Envy
  Inappropriate Intimacy
  Message Chains
  Middle Man
  ```

- [ ] **Step 3: Record validation judgment in the Korean content**

  The Korean document must explain these validated interpretation notes:

  ```markdown
  - 코드 스멜은 결함 판정표가 아니라 더 깊은 문제를 살펴보게 하는 신호다.
  - 숫자 기준은 질문을 시작하는 힌트이지 절대 규칙이 아니다.
  - DTO나 경계 모델처럼 Data Class가 적절한 자리도 있다.
  - 설명이 필요한 주석과 구조를 가리는 주석은 구분해야 한다.
  ```

### Task 2: Create Korean Wiki Document

**Files:**
- Create: `code-smells-ko.md`
- Read: `Code-Smells.md`
- Read: `junit-testcode-and-jacoco.md`
- Read: `testable-code-solid-legacy-refactoring.md`
- Read: `design-patterns-for-changeable-code.md`

- [ ] **Step 1: Create the Korean document skeleton**

  Add a Markdown document with these headings:

  ```markdown
  # 코드 스멜: 리팩토링이 필요한 신호를 읽는 법

  ## 1. 코드 스멜이란 무엇인가?
  ## 2. 코드 스멜을 볼 때 주의할 점
  ## 3. 코드 스멜 분류 한눈에 보기
  ## 4. Bloaters: 너무 커져서 다루기 어려운 코드
  ## 5. Object-Orientation Abusers: 객체지향을 어색하게 사용하는 코드
  ## 6. Change Preventers: 변경을 비싸게 만드는 코드
  ## 7. Dispensables: 없어지면 더 읽기 쉬운 코드
  ## 8. Couplers: 결합도를 높이는 코드
  ## 9. 레거시 코드에서 먼저 볼 코드 스멜
  ## 10. 코드 리뷰에서 사용할 질문
  ## 11. 정리
  ```

- [ ] **Step 2: Write opening guidance**

  The opening sections must state:

  ```markdown
  코드 스멜은 "이 코드는 틀렸다"라는 판정이 아니다.
  코드가 커지고 변경이 누적될 때 구조적 부담이 생겼는지 살펴보게 하는 신호다.
  ```

  Include the connection to earlier seminar topics:

  ```markdown
  테스트는 리팩토링 전에 동작을 보호하는 안전망이고,
  SOLID와 디자인 패턴은 스멜의 원인을 이해한 뒤 선택할 수 있는 도구다.
  ```

- [ ] **Step 3: Write the five category sections**

  For each smell, include:

  ```markdown
  ### [영문명] ([한글명])

  - 의미:
  - 살펴볼 질문:
  - 개선 방향:
  ```

  Keep each smell compact while covering every smell name validated in Task 1.

- [ ] **Step 4: Add legacy prioritization**

  Add a section that prioritizes:

  ```markdown
  1. 자주 바뀌는 God Class와 Long Method
  2. 장애나 버그 수정 때 여러 파일을 동시에 건드리게 하는 Shotgun Surgery
  3. 외부 객체 데이터를 과하게 끌어다 쓰는 Feature Envy
  4. 깊은 체인과 메시지 객체 전달이 남는 Message Chains
  5. 테스트 준비가 지나치게 많은 결합도 높은 코드
  ```

- [ ] **Step 5: Add review questions and summary**

  Include team review questions such as:

  ```markdown
  - 이 메서드는 한 화면을 넘어가면서도 흐름을 이해하기 쉬운가?
  - 이 변경 때문에 관련 없는 클래스가 함께 바뀌는가?
  - 데이터를 가진 객체보다 다른 객체의 필드를 더 많이 묻고 있는가?
  - 새 조건을 넣기 위해 같은 if/switch를 여러 곳에서 고쳐야 하는가?
  - 테스트를 쓰려면 외부 의존성을 너무 많이 준비해야 하는가?
  ```

### Task 3: Create Standalone Slide Deck

**Files:**
- Create: `slides/code-smells.html`
- Read: `slides/junit-testcode-and-jacoco.html`
- Read: `slides/design-patterns-for-changeable-code.html`
- Read: `code-smells-ko.md`

- [ ] **Step 1: Reuse the existing deck shell**

  Create a single HTML file with:

  ```html
  <main class="deck">
    <section class="slide active" data-note="..."></section>
  </main>
  <div class="progress"></div>
  <div class="counter"></div>
  ```

  Include the same presentation behaviors as the existing decks:

  ```text
  ArrowLeft / ArrowRight navigation
  touch previous / next navigation
  progress bar update
  slide counter update
  F fullscreen
  S speaker notes popup
  ```

- [ ] **Step 2: Compose 18 to 22 slides**

  Use this content flow:

  ```text
  01 Title
  02 Why this topic follows tests, SOLID, and patterns
  03 Smell is a question, not a verdict
  04 Five groups overview
  05 Legacy priority map
  06 Bloaters overview
  07 Long Method and Large Class example
  08 Primitive Obsession, Long Parameter List, Data Clumps
  09 Object-Orientation Abusers overview
  10 Switch Statements example
  11 Change Preventers overview
  12 Divergent Change versus Shotgun Surgery
  13 Dispensables overview
  14 Comments and Data Class nuance
  15 Couplers overview
  16 Feature Envy and Message Chains example
  17 From smell to refactoring question flow
  18 Team review checklist
  19 Summary
  ```

- [ ] **Step 3: Add visual teaching blocks**

  Include CSS-driven visuals instead of external assets:

  ```text
  five-group cards
  smell -> question -> small refactoring flow
  divergent change / shotgun surgery comparison
  message chain path diagram
  review checklist panel
  ```

- [ ] **Step 4: Add Java before/after snippets**

  Include compact snippets for:

  ```java
  public void process(TibrvMsg msg) {
      validate(msg);
      calculate(msg);
      save(msg);
      notify(msg);
  }
  ```

  and:

  ```java
  OrderRequest request = orderRequestMapper.from(msg);
  orderProcessor.process(request);
  ```

  Include a switch/strategy-oriented smell example without turning the deck into a Strategy pattern repeat.

- [ ] **Step 5: Add speaker notes**

  Every slide should have a concise `data-note` that gives the presenter one or two sentences of speaking guidance.

### Task 4: Verify Content And Presentation

**Files:**
- Verify: `code-smells-ko.md`
- Verify: `slides/code-smells.html`

- [ ] **Step 1: Check Markdown coverage**

  Run:

  ```powershell
  rg -n "Long Method|Large Class|Primitive Obsession|Long Parameter List|Data Clumps|Switch Statements|Temporary Field|Refused Bequest|Alternative Classes|Divergent Change|Shotgun Surgery|Parallel Inheritance|Comments|Duplicate Code|Lazy Class|Data Class|Dead Code|Speculative Generality|Feature Envy|Inappropriate Intimacy|Message Chains|Middle Man" code-smells-ko.md
  ```

  Expected: every source smell appears in the Korean document.

- [ ] **Step 2: Check slide deck presentation controls**

  Run:

  ```powershell
  rg -n "progress|counter|fullscreen|openNotes|touchstart|keydown|data-note" slides/code-smells.html
  ```

  Expected: the deck contains progress, slide counter, fullscreen, notes, touch, keyboard, and slide note logic.

- [ ] **Step 3: Check patch whitespace**

  Run:

  ```powershell
  git diff --check
  ```

  Expected: no whitespace errors.

- [ ] **Step 4: Browser-verify the slide deck**

  Open `slides/code-smells.html` in the local browser and verify:

  ```text
  first slide renders
  ArrowRight advances and ArrowLeft returns
  slide counter changes
  progress bar changes
  F enters fullscreen when browser allows it
  S opens speaker notes popup
  code and comparison layouts fit a desktop 16:9 frame
  ```

### Task 5: Review Git State

**Files:**
- Review: `code-smells-ko.md`
- Review: `slides/code-smells.html`

- [ ] **Step 1: Inspect the diff**

  Run:

  ```powershell
  git diff -- code-smells-ko.md slides/code-smells.html
  ```

  Expected: only the Korean seminar document and slide deck content appear.

- [ ] **Step 2: Inspect status**

  Run:

  ```powershell
  git status --short --branch
  ```

  Expected: the new intended artifacts are visible; unrelated untracked files such as `Code-Smells.md`, `refactoring-guru-catalog.md`, or older plan files remain unstaged unless the user asks to include them.
