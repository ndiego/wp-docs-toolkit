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
   Verified 2026-10-21: a copy of the demo blueprint with one added `installPlugin` step (`wordpress.org/plugins`,
   slug `gutenberg`, activate) boots in 14 s instead of 8 and the Gallery block then shows "Order by", so the
   blueprint route needs no Playground changes; the shot took 11 s against 3 s, with a network-idle warning, so a
   plugin suite may need a longer settle.
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
   covers it. Findings from a spike on 2026-10-21 (recordings and notes kept outside the repo):
   - **Build on Playwright's Screencast API, not Remotion.** Playwright 1.64, the version pinned here, has
     `page.screencast.start({ path, fps, size })` and `stop()`, a built-in animated pointer between actions
     (`showActions`, with a `style` option to hide its locator titles), overlays and chapter titles, and an `onFrame`
     callback with JPEG frames. Apache-2.0, already installed, works inside the Docker image. The spike recorded the
     Gallery block on the demo site at a steady 30 fps, 1920×1200 WebM VP9, 2.9 MB for 19 s, no dropped frames.
     Playwright's `recordVideo` also works (fixed 25 fps) but records from context creation, so it needs trimming.
   - **Remotion is ruled out as a dependency.** It is source-available under its own license: free for individuals,
     non-profits and companies of up to three people, and every company of four or more that runs the renders needs
     its own Company License (from $25 per seat per month). There is no open-source carve-out, so every handbook
     team that ran the toolkit's video renders would need a license, and the license's compatibility with
     GPL-2.0-or-later is unresolved. It also requires React and a webpack bundle step. What it would have added
     (a composited cursor, pacing, titles, encoding) the Screencast API and a small amount of code cover.
   - **Cursor and pacing come from the engine, not the recording.** The spike's injected cursor (a DOM arrow that
     follows eased `mouse.move` steps and shrinks on click) read better than Playwright's pointer, which moves
     linearly over about 0.3 s. Recording mode turns the existing verbs into eased moves with pauses; new verbs
     `moveTo`, `pause` and recording start and stop markers are the only vocabulary additions. Reduced motion must
     be off for the video context, unlike stills.
   - **Known limits.** Headless Chromium does not draw an open native `<select>` popup, so a dropdown shows the
     click and then the new value; container blocks such as Gallery need a visible "select parent" recipe because
     a click lands on an inner block; effects must exist in the target WordPress or plugin version (the Gallery
     block had no Duotone in 7.1.3 without the Gutenberg plugin). Trimming and transcoding to MP4 need an encoder:
     Playwright ships a cut-down ffmpeg, untested for this; ffmpeg-static is GPL-3.0-or-later.
   - **Review and hosting.** Step boundaries are captured as stills so the existing pixel diff reviews the
     keyframes; the video itself is reviewed by eye. Upload to VideoPress stays a manual step.
   - **Proof (2026-10-21).** An existing 22 s documentation video (Gallery block: Order by to A → Z, then Newest to
     oldest, then Duotone on all six images) was recreated step for step on the demo site with the Gutenberg
     blueprint, a six-image gallery created through `setup`-style REST calls, the injected cursor and the Screencast
     API: 20.9 s, 1920×1200, constant 30 fps, 9.2 MB WebM, reorder animation captured, 46 s per run including boot.
     Differences from the original were content and theme, the undrawn native select list, and the duotone preset,
     which had to be added to global styles because Ipsum defines only one. The step list in the engine's YAML
     vocabulary needed these additions: `moveTo` (eased, with a duration), `pause`, `record: start | stop`,
     `still`, `reducedMotion: false`, `fps`, and `setup` operations for attachment titles and dates and for a
     temporary global-styles change. That list is the specification for this item.
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
