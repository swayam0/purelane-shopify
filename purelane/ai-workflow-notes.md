# AI Workflow Notes — Purelane

## What I delegated
- **Structural audit** of the original HTML file (identifying the 5 required sections, flagging what wouldn't survive as-is in Shopify).
- **Writing the Liquid sections**, reusable snippets, and CSS for all 5 required sections, plus the custom multi-item add-to-cart web component for Bundles.
- **Step-by-step guidance** through Shopify Partner/dev-store setup, Shopify CLI, and git — since a lot of this was UI-driven, I had the AI narrate each click rather than write code for it.
- **A second agent (Antigravity)** working directly on my local filesystem placed the generated files at the correct paths and ran `shopify theme check` after each addition, reporting errors back.

## Where it broke or needed correction
1. **The nested Git submodule trap:** `shopify theme init` created its own nested `.git` inside the cloned Dawn folder, which caused my first commit to record the entire theme as a broken git submodule pointer instead of real tracked files. Caught this from the git warning output, not from any AI review — had to manually `rmdir` the nested repo and re-stage everything.
2. **Theme editor session confusion:** Two different theme editor sessions exist for a dev store (the persisted theme vs. the live CLI dev-session theme). I initially edited the wrong one twice, editing a theme that never received our custom sections. This wasn't caught until a screenshot showed the section list didn't match what had been built — worth automating a "confirm you're on the CLI dev theme" check earlier in the process.
3. **Ghost translation key:** One section referenced a translation key (`{{ 'x' | t }}`) that didn't exist in `locales/en.default.json`, caught cleanly by `shopify theme check` before it ever reached the browser.
4. **Scope creep:** I initially wrote an ambitious full-fidelity hero (scroll-driven parallax scene) but scoped it back to a functional static/CSS version once time pressure became clear — a case of catching my own overreach early rather than discovering it near the deadline.

## What I'd systematize for next time
- **Dev-theme confirmation:** A repeatable "confirm dev-theme session" checklist step immediately after `shopify theme dev` starts, before any editor work — the wrong-theme mixup cost real time twice.
- **Pre-flight Git check:** A standard check right after `shopify theme init`: verify there's exactly one `.git` at the outer repo root before the first commit, to catch the submodule trap before it happens rather than after.
- **Metaobject templates:** A shared metaobject-definition template (field names/types) for "bundle-like" content types, since Combos and Bundles turned out to need nearly identical modeling (title, price, compare price, linked products) with only minor differences.
- **Continuous Theme Checks:** Running `shopify theme check` immediately after *every* new file is placed, not just at natural checkpoints — it caught two real issues (the submodule structure indirectly, and the translation key directly) with zero manual review needed.
