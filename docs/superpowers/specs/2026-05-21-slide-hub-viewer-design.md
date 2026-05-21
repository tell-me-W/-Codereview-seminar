# Slide Hub Viewer Design

## Goal

Create a single entry HTML page for the three existing code-quality seminar slide decks.

The page should let a presenter choose one seminar session from a left-side table of contents while keeping the existing slide deck experience on the right.

## Current State

The repository already has three standalone HTML slide decks in `slides/`:

- `junit-testcode-and-jacoco.html`
- `testable-code-solid-legacy-refactoring.html`
- `design-patterns-for-changeable-code.html`

Each deck owns its own presentation controls, including slide navigation, progress, speaker notes, and fullscreen controls.

## Chosen Approach

Add a hub page at `slides/index.html`.

The hub page will use:

- A left navigation panel with exactly three seminar session entries
- A right content area with an `iframe`
- The existing standalone HTML slide deck files as iframe targets

The default selected session is `JUnit Test Code와 JaCoCo`.

This approach preserves the existing deck HTML files and avoids merging three independent slide engines into one large document.

## Interaction Design

### Left Navigation

The left panel shows only session-level entries:

1. `JUnit Test Code와 JaCoCo`
2. `테스트하기 쉬운 코드: SOLID와 레거시 리팩토링`
3. `디자인 패턴: 변경에 강한 코드를 만드는 반복 해법`

The selected item has a clear active state.

### Right Viewer

The right pane renders the selected deck in an iframe.

Selecting another session replaces the iframe target with the corresponding standalone deck file.

The hub does not add slide-level navigation, deck playback controls, or a duplicated fullscreen/notes toolbar. The embedded deck keeps its existing controls and keyboard handling.

## Visual Design

The hub should fit the existing corporate slide tone:

- Light background
- Restrained navy and gold accents
- Clear seminar/session hierarchy in the left panel
- Large right viewer area with minimal framing

Desktop layout uses a fixed left sidebar and a dominant right viewer.

On narrow screens, the session navigation should compact above or alongside the viewer so the selected deck remains usable.

## Boundaries

In scope:

- New `slides/index.html` hub page
- Session selection UI
- Iframe viewer targeting the existing three deck files
- Responsive handling for the navigation/viewer split

Out of scope:

- Rewriting or merging the three existing slide decks
- Adding per-slide table of contents
- Adding hub-level slide controls
- Converting the decks to PPTX

## Verification

Check that:

- `slides/index.html` loads the JUnit deck by default
- Each left navigation item loads the correct existing HTML deck
- Active navigation state updates with the selected session
- Existing deck controls remain usable inside the iframe
- Narrow and desktop layouts leave the viewer readable
- Existing slide deck source files are referenced rather than structurally merged
