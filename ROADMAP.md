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

## Rules locked in

- **Motion is a video, never an animated image** (2026-10-21). Handbooks use static images or hosted video. GIFs and
  animated WebP autoplay and loop without a pause control. The engine's reserved `kind: clip` therefore means a
  short video, and nothing in the toolkit will ever write an animated image. The full rule is in
  `style/CONVENTIONS.md`; when a sequence of stills is enough is in `style/SCREENSHOTS.md`.

## Where capture stands (2026-10-21)

Taking one screenshot is established: a 3 s capture after an 8 s boot, deterministic pixels through the pinned
Docker image, reduced motion, disabled animations, hidden caret, network idle plus a settle delay, masks for dynamic
regions, seven presets, seven scenes, twenty-five verbs, twenty-one targets, locators by accessible name, setup
fixtures and `inspect`. 214 unit tests cover the spec loader, locators, targets, presets, image pipeline, manifest and
lint. What is not established is surviving versions and volume; those items are in the list below.

## What is missing, in the order it should be built

1. **An integration run against the next WordPress.** All 214 tests use a fake page; the example project is the
   only end-to-end check and a person runs it. The parts most likely to break on a release, the Gutenberg helpers
   ported into `src/wp.js` and the accessible names presets depend on, have no automated coverage. A scheduled job
   that renders `examples/block-editor-handbook` against the WordPress beta or nightly (Playground accepts those
   through `--wp`) would show preset breakage weeks before release day. Do this before any handbook adopts.
2. **Required plugins, declared where the shot is** (asked for 2026-10-21). Some screenshots document features
   that live in a plugin: the Gutenberg plugin ahead of a WordPress release (the Gallery block's "Order by" control
   is one; WordPress 7.1.3 still shows a "Randomize order" toggle), the MCP adapter plugin, or any plugin a handbook
   documents. The user or the spec names the plugins; the engine provides them. Design:
   - A `plugins` list at three levels, merged: the project's `shots.config.js`, a spec file, a shot. Each entry is a
     wordpress.org slug with an optional pinned version (`gutenberg`, `gutenberg@22.3.0`), a zip URL, or a local path,
     so plugins that are not on wordpress.org (a GitHub release zip, a plugin under development) work too. Same list
     as `--plugin` on `shoot` and a `plugins` parameter on the MCP `screenshot` and `inspect` tools.
   - Plugins change the whole environment, so they are installed at boot through the blueprint's `installPlugin`
     step, not per shot through `setup`. The runner already boots once per suite; it would group shots by suite plus
     the sorted plugin set and boot once per group, keeping unrelated shots on the clean site. `setup`'s `plugin`
     operation stays for toggling an installed plugin's active state within a shot.
   - The resolved plugin list with versions goes into the shot hash and the manifest entry next to `wordpress`, so
     a plugin update is a re-render and a review like a WordPress update. Unpinned entries resolve to the version
     installed at render time and record it.
   - The demo site ships a `gutenberg` preset for the common case; projects add named plugin sets of their own.
   What exists today: `suite:` on a spec selects `<suitesDir>/<name>/blueprint.json` and boots once per suite, but
   no suites ship and a suite needs a whole blueprint; `setup`'s `plugin` only activates an installed plugin.
3. **A retry before reporting a diff.** The runner has no retry or flake handling; one slow response or popover
   race fails that shot, and at a thousand shots a half-percent flake rate is five false diffs per run that the
   report cannot tell from real changes. Re-render a failed or changed shot once before staging it.
4. **A screenshots action and an example workflow.** Nothing runs the loop yet. The repository has a CI workflow
   for itself and an action for the validator, but no action that runs `check --docker` in a handbook, no scheduled
   job and no example workflow in `examples/block-editor-handbook`. The release run is a person at a terminal with
   Docker. Canonical pixels come only from the pinned Playwright image, so the runner needs Docker; decide where the
   job runs before writing the action. Deferred on purpose (2026-10-21): not needed until a handbook has specs worth
   re-rendering.
5. **A coverage command.** The engine re-renders only what has a spec, and the existing documentation has thousands
   of screenshots and no specs. Backfilling them is the real adoption cost. A per-handbook report of article images
   that are not engine-generated turns the backfill into a tracked number. The MCP plus `inspect` is the authoring
   tool for it.
6. **A version-bump procedure.** The WordPress version is pinned in the demo site blueprint and again in each
   project's `shots.config.js`. A release means a toolkit release with a new blueprint, then each handbook bumps its
   pin and runs `check`. Write it down once it has been done for real; the first release cycle will show which
   presets survive a UI change and which need accessible-name fallbacks.
7. **More showcase blocks.** Image, Gallery and Table have demo content; every other block needs `post: new` or
   hand-written steps. Add blocks as the first handbook asks for them.
8. **A timed run at realistic scale, then parallel sessions.** The example project has fifteen shots. A handbook is
   hundreds and the whole of WordPress.org is thousands. One Playground, one browser, one shot at a time; shots
   marked `isolated` reboot Playground. Measure a large run before promising release-day turnaround. Stabilisation is
   heuristic (a 250 ms settle, an 8 s network-idle cap) and tuned on a developer machine; a loaded CI runner may
   produce occasional diffs from lazy images or skeleton states, which masks and the retry in item 3 should absorb.
   Verify under load.
9. **Video from the engine.** Articles that show a transition (a drag, a reorder animation, a multi-step flow) need
   a hosted video, and the engine should produce it from the same scene and steps as a still so a release re-render
   covers it. The sketch: a recording mode runs the steps with eased mouse movement and a visible cursor, captures
   frames at a fixed rate through the CDP screencast inside the Docker image, and writes a timeline (step boundaries,
   mouse path, clicks); a Remotion composition turns frames plus timeline plus style tokens into an MP4 with a
   consistent frame, cursor and pacing; the step boundaries are also captured as stills so the existing pixel diff
   reviews the keyframes. Upload to VideoPress stays a manual step. Open questions: how a changed video is reviewed,
   and whether Playwright's own `recordVideo` is good enough for a first version.
10. **Prose accuracy.** The toolkit keeps images accurate; it does not yet check that a control an article names
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
