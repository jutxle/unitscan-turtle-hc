# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

UnitScan-Turtle-HC is a World of Warcraft addon for Turtle WoW (Mysteries of Azeroth, 1.12.2 client). It scans for dangerous elite mobs, rares, and hostile NPCs nearby and alerts the player with an animated popup. Designed for hardcore mode where death is permanent.

Fork of [unitscan-turtle](https://github.com/GryllsAddons/unitscan-turtle) with a prepopulated zone-specific mob database.

## Development

**Language:** Lua (WoW 1.12.2 Addon API)
**No build system, tests, or CI.** Edit Lua files directly; reload the addon in-game.

To install/test: copy the addon folder to `TWMOA_1171/Interface/AddOns/unitscan-turtle-hc/`.

## Architecture

Two Lua files, loaded in order by the TOC file:

- **zonetargets.lua** — Data-only file. Single function `unitscan_zone_targets()` returns a table of mob names keyed by zone. Uses a long if/elseif chain on zone name. ~40 zones covered including dungeons and custom Turtle WoW zones (Gilneas, Tel'Abim, etc.).

- **unitscan.lua** — All addon logic and UI in one file. Key components:
  - **Event handler** (`unitscan.onevent`) — Dispatches WoW events (VARIABLES_LOADED, PLAYER_REGEN_ENABLED/DISABLED, MINIMAP_ZONE_CHANGED, START/STOP_AUTOREPEAT_SPELL) to named handler functions.
  - **Scanner** (`unitscan.UPDATE`) — Runs on a 1-second timer. Calls `unitscan.check_for_targets()` which iterates through zone targets + custom user targets, using `TargetByName()` to detect nearby units. Saves/restores the player's original target.
  - **Alert UI** (`unitscan.LOAD`) — Creates an achievement-style popup frame with a 3D model viewer, glow/shine animations, flash overlay, and close button. Movable via CTRL+SHIFT+LEFT CLICK drag.
  - **Scanning pauses** automatically during combat and while auto-attack/auto-shoot/wand is active.

## Key Patterns

- **SavedVariables:** `unitscan_targets` (declared in TOC) persists the user's custom target list across sessions.
- **Zone reload:** Zone targets refresh on zone change (`MINIMAP_ZONE_CHANGED`) and 90 seconds after finding a target (to re-detect roaming mobs).
- **Target safety:** Scanner saves current target before `TargetByName()` calls and restores it afterward. Does NOT auto-target found units — player must click the alert or use `/unitscantarget`.

## Slash Commands

- `/unitscan` — Lists active scan targets
- `/unitscan <name>` — Toggles a custom target on/off
- `/unitscantarget` — Targets the most recently found unit

## TOC File Format

`unitscan-turtle-hc.toc` declares addon metadata (Interface version 11200, SavedVariables, load order). When adding new Lua files, they must be listed here.
