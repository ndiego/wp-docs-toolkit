# Roadmap

Where the toolkit is going and what stands between the current release and the end goal. Written 2026-10-21 at
0.4.0; update it when a milestone lands or the goal changes. `CHANGELOG.md` records what shipped; this file records
what has not.

## End goal

When a WordPress release ships, run the whole of the WordPress.org documentation against the toolkit and get back
a set of screenshots to review, with the handbooks' prose checked the same way. Routinely, as a job rather than a
project. The same engine serves ad hoc work: a review of one article, or one screenshot in the house style from a
terminal or a prompt.

All WordPress.org documentation is moving to Git in a separate project, so every handbook, HelpHub included, will
use the same technical flow: Markdown and specs in a repository, the `wporg-markdown` importer publishing from it.
The toolkit assumes that flow and nothing else.

## What 0.4.0 already does for that goal

- **Specs are subjects, not scripts.** A shot names a preset and its parameters; the preset supplies scene, steps and
  framing. When a release moves a control, the fix is in one preset and every spec that uses it follows.
- **A release is a diff.** Every manifest entry records WordPress, PHP, theme, Playwright and Chromium versions, the
  blueprint hash and a pixel hash. `check --docker` re-renders, compares, stages only what changed and writes a
  report; `update` accepts the staged images.
- **Ad hoc work goes through the same presets and style.** `wp-screenshots shoot` from a terminal and the MCP
  `screenshot` tool from a prompt render the same frames, tokens and annotation rules, and the MCP returns the spec
  to commit. The `--article` and `--shot` filters give the per-article review.
- **The manifest validator checks what the importer does**, so a change that would break an import fails before it
  merges.

## What is missing, in the order it should be built

1. **A screenshots action and an example workflow.** Nothing runs the loop yet. The repository has a CI workflow
   for itself and an action for the validator, but no action that runs `check --docker` in a handbook, no scheduled
   job and no example workflow in `examples/block-editor-handbook`. The release run is a person at a terminal with
   Docker. Canonical pixels come only from the pinned Playwright image, so the runner needs Docker; decide where the
   job runs before writing the action. Deferred on purpose (2026-10-21): not needed until a handbook has specs worth
   re-rendering.
2. **A coverage command.** The engine re-renders only what has a spec, and the existing documentation has thousands
   of screenshots and no specs. Backfilling them is the real adoption cost. A per-handbook report of article images
   that are not engine-generated turns the backfill into a tracked number. The MCP plus `inspect` is the authoring
   tool for it.
3. **A version-bump procedure.** The WordPress version is pinned in the demo site blueprint and again in each
   project's `shots.config.js`. A release means a toolkit release with a new blueprint, then each handbook bumps its
   pin and runs `check`. Write it down once it has been done for real; the first release cycle will show which
   presets survive a UI change and which need accessible-name fallbacks.
4. **More showcase blocks.** Image, Gallery and Table have demo content; every other block needs `post: new` or
   hand-written steps. Add blocks as the first handbook asks for them.
5. **A timed run at realistic scale.** The example project has fifteen shots. A handbook is hundreds and the whole
   of WordPress.org is thousands. Shots marked `isolated` reboot Playground, and nothing renders in parallel. Measure
   a large run before promising release-day turnaround; parallel sessions are the likely fix.
6. **Prose accuracy.** The toolkit keeps images accurate; it does not yet check that a control an article names
   still exists with that name. `inspect` already returns the accessible names on screen, so a check that
   cross-references an article's named controls against the UI reuses what exists. The Markdown conventions
   linter and a WordPress.org style and spelling check (a Vale package plus a skill; the Docs team style guide stays
   the source of truth) belong to the same layer.

Also open, from the README: a hosted MCP endpoint, and an upstream request to slim the Playground runtime, which
ships a PHP build for every version.

## Principles that constrain the roadmap

The ones in the README hold: nothing here knows a handbook's slugs, paths or URLs; nothing changes how
developer.wordpress.org renders; consumers pin a version; maintenance tooling only, migration tooling lives in
[wp-documentation-migration](https://github.com/ndiego/wp-documentation-migration).
