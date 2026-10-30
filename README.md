# Mega Man X4: Axl 3P Co-op

> **Status: pre-alpha planning fork. There is no playable Axl build yet.**
>
> This fork explores adding **Axl** as the third playable character in a
> **3-player-only** Mega Man X4 co-op build. It is built on
> [mstan/MegaManX4Recomp](https://github.com/mstan/MegaManX4Recomp), the native
> PC recompilation of Mega Man X4 USA (SLUS-00561).

**P1: X · P2: Zero · P3: Axl**

Axl is planned as a **hybrid X/Zero** character: X's gunplay and native body,
Zero's double jump, and Axl's dodge roll. **No saber. No Copy Shot. No Ultimate
Armor.** Axl is a guest character; P1 still owns the campaign and story.

[**Read the full plan and decision gates →**](https://github.com/YossefKhalil/MegamanX4-Axl-Port/issues/1)

---

## What works today

| Piece | Status |
| --- | --- |
| Upstream 2P X/Zero co-op | **Stable**, upstream `1.0.19` public beta |
| 3P co-op code | **Draft PR #69**, experimental, not merged into this fork |
| Axl in this fork | **Not implemented** |
| Local build | **Blocked**: toolchain and submodules are not set up yet |
| Disc verification | **Passes**: USA disc SHA-256 matches `game.toml` |
| Axl sprite sheet | **Available**; fan compilation, not production-ready |

This repository currently contains the normal 2P co-op code, not the 3P code.
The first engineering task is to integrate the 3P baseline from
[PR #69](https://github.com/mstan/MegaManX4Recomp/pull/69) into a dedicated
`axl/3p` branch and strip it down to 3P.

---

## The planned Axl

Axl is not a clone of X or Zero. The target is a character who feels like Axl
while reusing as much native X4 logic as possible.

| Ability | v1 plan | Why |
| --- | --- | --- |
| **Buster / rapid fire** | X-style buster with faster cooldown or hold-to-fire | The sprite sheet is gun-centric; X's shooting pipeline is native |
| **Double jump** | Zero's double jump, patched for Axl | Cheapest way to add Zero's mobility without a saber |
| **Dodge roll** | Axl roll animation with brief invincibility, reusing the dash state | The sheet has a full dodge-roll row |
| **Hover** | Deferred to v1.1 | Native hover exists, but combining it with double jump needs input/state design |
| **Saber** | **No** | The sheet has no Zero saber combo frames |
| **Copy Shot** | **No** | Separate enemy-DNA/transform subsystem; total-conversion scope |
| **Ultimate Armor** | **No** | No Axl Ultimate design or palette in the sheet |
| **Charge shot** | **No in v1** | No charged-buster poses in the sheet |
| **Boss weapons** | **No in v1** | Keeps animation and balance scope bounded |
| **Unique menus/voice** | **No in v1** | Reuse X's menu/UI and voice bank; polish later |

### Design rules

- Axl is a **guest**: P1 owns story, bosses, doors, and permanent progress.
- Add **one ability at a time**; do not ship buster + saber + double jump +
  air dash + hover together.
- Reuse native systems before writing new ones.
- Test **3P local first**, then LAN netplay.
- Keep Axl assets **external**; do not bundle fan or Capcom-derived art.

---

## Why 3P only?

The 4-player mode in PR #69 is an impressive experimental prototype, but it is
not the right base for a character project:

- The 1 MiB guest mod-memory budget is nearly full.
- 4P carries extra camera, HUD, texture-cache, and netplay complexity.
- 3P is the smallest configuration that delivers the requested experience.
- A focused 3P build is easier to stabilize and test.

The memory plan is:

| Slot | Contents | Size |
| --- | --- | --- |
| `assets[0]` | X, full menus/UI | `0x58000` |
| `assets[1]` | Zero, full menus/UI | `0x58000` |
| `assets[2]` | Axl sprites + assembly + colors + palette; reuse X menus/UI/audio | `0x42000` |
| diagnostic | existing | `0x1000` |
| **Total** | | **`0xF3000`** (~972 KiB of 1 MiB) |

Axl must fit in the same hole currently used by Ultimate Armor X's spare
assets. If the sheet's sprite body exceeds `0x34000` after compression, we drop
effect frames, share X effects, or change the plan. That is a **hard gate**.

---

## Roadmap

Estimates assume one developer plus AI assistance, with the user running builds
and playtests. They are planning ranges, not promises.

| Phase | Work | Gate |
| --- | --- | --- |
| **0. Baseline** | Install toolchain, init submodules, verify disc, build 2P; merge PR #69 into `axl/3p`; build 3P | P1 X + P2 Zero + P3 Ultimate X spawn and move |
| **1. Asset spike** | ARC reader/writer, LZ codec, sheet slicer, palette converter, minimal Axl ARC | Axl idle renders in-game |
| **2. Third slot** | Add `MMX4_HERO_AXL`, asset slot 2, fixed roster; repurpose `spare_x`; remove Ultimate Armor | P3 Axl spawns; memory stays under 1 MiB |
| **3. Hybrid movement** | Double jump, roll, buster behavior, HUD/menu sanity | One stage with movement, shooting, double jump, roll, no softlock |
| **4. Animation/gameplay** | Full state coverage, assembly mapping, bosses, death/respawn, transitions | 2–3 stages + a boss, local 3P |
| **5. Netplay/campaign** | 3P handshake, campaign doors/cutscenes/vehicles, performance, balance | 3P LAN campaign run with no desync/softlock |

**Rough total:** 3–6 months for a solid hybrid, 6–9 months if the assembly
format must be reverse-engineered from scratch. A polished X reskin is 2–6
weeks; an Ultimate Armor reskin is 3–8 weeks.

---

## Current blockers

1. **No build toolchain.** CMake, Ninja, and a C/C++ compiler are not installed;
   `psxrecomp-v4` and `recomp-ui` submodules are not initialized.
2. **3P code is unmerged.** PR #69 has 61 changed files and conflicts with
   current `master`, including `mmx4_coop_plugin.c` and the framework pin.
3. **No assembly compiler.** X4's assembly member must be parsed or patched by
   hand. This is the largest unknown.
4. **Memory is tight.** Axl must fit `0x42000` or the clean plan changes.
5. **No dedicated artist.** The sheet covers many states but is a compilation,
   not a production animation set. Missing frames will be reused or cut.

The detailed edge-case list, hard/soft limits, and fallback order are in
[issue #1](https://github.com/YossefKhalil/MegamanX4-Axl-Port/issues/1).

---

## Build from source

Building requires the original USA disc, CMake, a C/C++ toolchain, Python, SDL2,
and OpenGL. Initialize submodules first:

```sh
git clone --recurse-submodules https://github.com/YossefKhalil/MegamanX4-Axl-Port.git
cd MegamanX4-Axl-Port
```

Place your own verified `.cue` + `.bin` under the ignored `mmx4/` directory.
Do not convert the disc to a cooked ISO; X4 uses XA sectors.

Build instructions and framework prerequisites are in
[docs/DEVELOPMENT.md](docs/DEVELOPMENT.md). The upstream project is the source
of truth for the recompilation framework and release packaging.

> This fork is not yet buildable out of the box. That is a setup task, not a
> statement about the upstream project.

---

## Assets and legal

- **Code:** original project code is [MIT licensed](LICENSE); dependencies keep
  their own licenses and notices.
- **Game:** Mega Man X4 and its imagery belong to Capcom. You supply your own
  original, unmodified USA disc. This is an unofficial fan project.
- **Axl sprite sheet:** the PS1-style Axl sheet is a fan compilation by
  [JuggaloManX](https://www.deviantart.com/juggalomanx/art/Ultimate-Axl-Sprite-Sheet-%282021%29-824018313)
  and contributing spriters. It is **not bundled** with this repository.
  Permission and credit will be requested once there is a demo.
- **No Copy Shot / no story rewrite:** Axl is a guest character, not a
  replacement for X or Zero.

---

## Credits

- [mstan/MegaManX4Recomp](https://github.com/mstan/MegaManX4Recomp) — the native
  X4 recompilation and the stable 2P co-op baseline.
- [mstan](https://github.com/mstan) — PR #69's experimental 3P/4P framework.
- [sozud/mmx4](https://github.com/sozud/mmx4) — the WIP X4 decompilation used
  for character/state-machine research.
- [JuggaloManX](https://www.deviantart.com/juggalomanx) and the Axl sheet
  contributors — the 2D Axl art.
- [PSXRecomp](https://github.com/mstan/psxrecomp) and
  [recomp-ui](https://github.com/mstan/recomp-ui) — the recompilation framework.

[**Planning issue #1**](https://github.com/YossefKhalil/MegamanX4-Axl-Port/issues/1) ·
[Upstream project](https://github.com/mstan/MegaManX4Recomp) ·
[PR #69](https://github.com/mstan/MegaManX4Recomp/pull/69)
