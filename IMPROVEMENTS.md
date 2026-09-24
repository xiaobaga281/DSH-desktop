# What this build changes over the base

DeepSeek Harness ships as a command-line plugin kernel. This build is a personal fork that turns it into a Windows desktop product, and the fork is maintained on top of the base rather than as a separate code line. This page lists what was added or changed, grouped by what a reader can do differently, and gives the commands that re-derive every number here.

## Table of contents

- [Reading the counts](#reading-the-counts)
- [Desktop shell and packaging](#desktop-shell-and-packaging)
- [Agent teams](#agent-teams)
- [Session council](#session-council)
- [Computer and browser control](#computer-and-browser-control)
- [Memory and decisions](#memory-and-decisions)
- [Product surface and Chinese UI](#product-surface-and-chinese-ui)
- [Media and long video](#media-and-long-video)
- [Upgrade machinery](#upgrade-machinery)
- [Known gaps](#known-gaps)

## Reading the counts

Measured on 2026-09-24 against a pristine copy of the same base version: 11,239 base files become 11,861, of which 440 are modified base files and 622 are added here. Nothing is deleted. The work is filed under 40 capability surfaces, and each surface declares its own merge policy and verification step.

Re-derive them from a source checkout of the fork:

```powershell
python product-layer\tools\product-layer-verify.py
python -c "import json;m=json.load(open('product-layer/capsule/manifest.json',encoding='utf-8-sig'));print(m['counts'],m['blobFiles'],len(m['surfaces']))"
```

## Desktop shell and packaging

113 paths. The base has no window; this build owns one.

- A native window with its own traffic-light chrome, including the boot surface painted before the frame mounts so the controls are never stranded over an empty page.
- A per-user Inno Setup installer that needs no administrator rights, plus a portable package layout whose dependency links are rebuilt on first launch instead of pointing at the machine that produced them.
- Windows notification identity that names the finished task in the toast, and a notification sound the renderer plays because the packaged app cannot select a system sound by name.
- The icon and the four-part Windows version resource are embedded into the shipped executable.

## Agent teams

163 paths. The base can run subagents; this build runs them as a team you can watch.

- Built-in team templates, an office view, and studio settings for the members.
- A plan budget that gates one stage at a time, with an owner recorded for every stage.
- One agent-preset menu across the surface, and a subagent panel that shows what each member is doing.

## Session council

18 paths. Two independent sessions negotiate instead of overwriting each other's files.

- One durable board plus the `peer_*` tools, so a sibling session can post, lease, and answer.
- The tools mount only when a live sibling exists, so a lone session never sees a control it cannot use.

## Computer and browser control

118 paths. The base reaches the network; this build reaches the screen.

- A resident desktop driver for pointer and screen access, measured at 2 ms per pointer read and 316 ms per capture on the test machine.
- A control indicator that shows which member is moving the machine, with the states a user can read without a legend.
- An in-app browser panel with provider depth, and the web search and fetch tools adapted to it.

## Memory and decisions

178 paths, of which 162 are decision records.

- Distillation, retrieval, and consolidation for durable memory.
- Agent Notes as a first-class part of the repository: each non-trivial change carries why it was made, what it rejected, and what it cost.

## Product surface and Chinese UI

299 paths. The whole surface was localized and reorganized rather than translated in place.

- A settings shell with reordered sections, model catalog fetch, plugin and skill management, an archive page with permanent-delete semantics, and an Operations page that collects the policies of 16 `computer_*` tools in one place.
- Themes, background images, a desktop pet, and a hero and brand pass over the first screen.
- A skill and MCP marketplace. Measured on the shipped build it reads 12 built-in and 20 GitHub skills plus 165 MCP registry entries; a skill install lands in `dsh-home\skills` and an MCP install is written into a managed block of `cordis.patch.yml`.
- Installs take effect without a restart: the desktop profile watches its user patch layer, so the next turn can call the new server. The panel states what its own check can prove, which is that the row is in the config file.

## Media and long video

18 paths. Media tools and a long-video team that plans by form - picture, narration, cut rhythm - instead of a hard-coded clip length.

## Upgrade machinery

147 paths. This is the part that keeps every section above from being thrown away at the next base release.

- A self-contained capsule: every modified base file is stored together with its pristine base original as a content-addressed blob, so upgrading needs only the new official source and the capsule. No previous clean checkout, no hand merge.
- Three registration checks that answer different questions: does the tree match the capsule, does the human-readable ledger list every product path, and does each declared path still exist.
- One upgrade driver that refuses to copy a single file while the capsule and the ledger disagree, and can re-baseline the capsule onto the new version in the same run.

## Known gaps

- Two declared surfaces, `ptc-runtime` and `typert-generator`, currently hold zero paths: the classification is written, the customization is not.
- The counts on this page move whenever the fork adds a decision record, because those files are part of the product layer. Re-run the commands above instead of trusting a quoted number.
- The marketplace reports what its own check can prove. It does not yet offer a per-server reconnect or log view, which is where a configured-but-unreachable server stops being diagnosable from the interface.

## Dev Note

Every number here was read from `product-layer/capsule/manifest.json` after a capture on 2026-09-24, or measured in a running instance of the shipped build. The desktop-driver timings come from one x64 machine and will differ elsewhere. The marketplace behavior was verified by installing a skill and an MCP server through the shipped interface, not by reading the code alone.
