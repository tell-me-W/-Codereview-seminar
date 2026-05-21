# Code Smells Seminar Design

## Goal

Validate the existing `Code-Smells.md` catalog summary, create a Korean Wiki-style seminar document, and create a presentation-ready HTML slide deck that matches the existing code-quality seminar decks.

## Current State

The repository already contains:

- An untracked English source summary at `Code-Smells.md`
- Three Korean code-quality seminar Markdown documents
- Three standalone seminar slide decks in `slides/`
- A slide hub at `slides/index.html`

The existing `Code-Smells.md` is a compact catalog organized around the common code-smell groups:

- Bloaters
- Object-Orientation Abusers
- Change Preventers
- Dispensables
- Couplers

## Validation Direction

Validation should compare the source summary against established references for code smells and refactoring.

The work should:

- Check that the major smell groups and listed smell names are not accidentally altered or omitted
- Note where the source is a compact summary rather than a complete teaching document
- Correct misleading absolutes in the Korean material
- Emphasize that a code smell is a signal to investigate, not automatic proof that code is wrong

The Korean material should add practical judgment around smells that can be over-applied, especially comments, data classes, long methods, and abstraction-heavy fixes.

## Chosen Outputs

Create two new seminar artifacts while leaving the English source unchanged:

- `code-smells-ko.md`
- `slides/code-smells.html`

The Markdown file is a Wiki-style reading document.

The HTML file is a standalone presentation deck for the seminar and should fit the existing slide tone and behavior.

## Korean Markdown Design

The Korean Markdown should be suitable for internal Wiki distribution and should not be a line-by-line translation only.

Recommended structure:

1. What a code smell is
2. How to interpret a smell without turning it into a rigid rule
3. The five smell groups
4. Each smell with meaning and refactoring questions
5. Smells to inspect first in legacy code
6. Review questions the team can use
7. Summary

The writing should connect code smells to the team's prior seminar topics:

- Tests help protect behavior before refactoring
- SOLID and patterns are possible tools after a smell is understood
- Legacy refactoring should start from risky, frequently changed, or hard-to-test areas

## Slide Deck Design

The slide deck should be concise enough for a seminar presentation instead of copying the Markdown in full.

It should:

- Use the visual language already established by the other `slides/*.html` decks
- Be a standalone single HTML presentation deck
- Include speaker notes
- Include slide progress, counter, keyboard navigation, fullscreen, and notes popup behavior consistent with the existing decks
- Use diagrams, tables, and Java-flavored before/after snippets instead of external images

Recommended flow:

1. Why code smells matter after tests, SOLID, and patterns
2. Smell is a question, not a verdict
3. Five smell groups at a glance
4. Legacy priority smells such as Long Method, Large Class, Shotgun Surgery, Feature Envy, and Message Chains
5. A few focused examples that move from smell to refactoring question
6. Review checklist and team application

The target size is roughly 18 to 22 slides.

## Validation Notes To Surface

The seminar artifacts should explicitly surface these interpretation notes:

- A long method can be a useful smell without every method over a numeric threshold being automatically bad
- Comments are not inherently harmful; comments that compensate for unclear structure can reveal a design or naming problem
- Data classes can be a smell in object-oriented modeling, but DTOs and boundary models may still be appropriate
- A smell should guide investigation before choosing a refactoring

## Boundaries

In scope:

- Validate `Code-Smells.md` against references
- Create one Korean Wiki-style Markdown document
- Create one seminar slide deck HTML under `slides/`
- Keep style aligned with the existing seminar materials

Out of scope:

- Rewriting the existing English source file
- Adding the new deck to the slide hub unless requested later
- Creating PPTX output
- Turning every catalog item into a full refactoring recipe

## Verification

Check that:

- The Korean document covers the five smell groups from the source
- The translated smell names and explanations are internally consistent
- The deck opens as a standalone HTML file
- Existing deck controls and visual tone are matched closely
- Code samples and diagrams fit a 16:9 desktop presentation frame
- The worktree only stages the intended new artifacts and approved spec changes
