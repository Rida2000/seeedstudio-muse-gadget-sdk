<!--
Copyright (c) Meta Platforms, Inc. and affiliates.

Licensed under the Apache License, Version 2.0 (the "License");
you may not use this file except in compliance with the License.
You may obtain a copy of the License at

    http://www.apache.org/licenses/LICENSE-2.0

Unless required by applicable law or agreed to in writing, software
distributed under the License is distributed on an "AS IS" BASIS,
WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
See the License for the specific language governing permissions and
limitations under the License.
-->

# AGENTS.md

Start here when you work in this repository with a coding agent. This is
Seeed Studio's copy of the open source
[Muse Gadget SDK](https://github.com/facebookincubator/muse-gadget-sdk): the
firmware and tools that turn a board into a Muse gadget, plus support for
Seeed's own devices. People use it as the sample project for their own Muse
apps and demos on Seeed hardware.

This file is the map. The details live next to the code:

| Read | For |
|---|---|
| [`esp32/AGENTS.md`](esp32/AGENTS.md) | ESP-IDF setup, building, identifying a board, flashing, monitoring, pairing, adding a command, tests |
| [`esp32/devices/AGENTS.md`](esp32/devices/AGENTS.md) | Porting a board: vendor sources, the overlay, the `muse_board_t` contract, docs and verification |
| [`linux/AGENTS.md`](linux/AGENTS.md) | The Linux Device SDK (Raspberry Pi and other Linux boxes) |
| [`esp32/devices/README.md`](esp32/devices/README.md) | Every board's hardware, features and build, and what has been checked on hardware |
| [`skills/README.md`](skills/README.md) | Markdown skills that teach Muse to use other devices through a gadget |
| [CodeCraft Hub guide](README.md#share-your-project-on-codecraft-hub) | Sharing a project for one-click firmware deployment; required for the Seeed Studio Muse Gadget Challenge |

## Repository map

| Path | What's there |
|---|---|
| `esp32/main/` | Home Link: Wi-Fi, BLE pairing, the encrypted session to Muse, the commands Muse can call, status lights and screens |
| `esp32/components/muse/` | The on-screen UI, voice, settings and one board file per UI board in `boards/` |
| `esp32/devices/` | One `sdkconfig.<board>` overlay per board, and the device docs |
| `esp32/tools/` | `board.sh` (status-light and status-screen boards), `muse/board.sh` (UI boards), `muse/monitor.py`, `muse/chat.py`, `muse/snap.py`, `muse/avatar.py`, `image_for_display.py` |
| `esp32/tests/` | Host tests: they compile firmware code against fakes, no board needed |
| `esp32/simulator/` | The UI on the desktop, without a board |
| `android/musegadgetstest/`, `ios/musegadgetstest/`, `protocols/VIDEO.md` | Phone camera streaming to supported ESP32 screens |
| `web-flasher/` | The browser installer; `boards.json` lists its boards |
| `linux/`, `skills/` | The Linux SDK, and device skills for Muse |
| `README.md`, `README.zh-CN.md` | The beginner's guide to Seeed devices, in English and Chinese |

## Seeed devices

| Device | Profile (helper alias) | Board code | Console USB | Status |
|---|---|---|---|---|
| SenseCAP Watcher | `tools/muse/board.sh … watcher` | `components/muse/boards/board_sensecap_watcher.c` | CH342 `1a86:55d2`, the second port | Upstream |
| SenseCAP Indicator | `tools/board.sh sensecap-indicator …` | `main/led_status.c`, `main/sensecap_sensors.c` | CH340 `1a86:7523` | Upstream |
| reTerminal E1001 / E1002 | `tools/board.sh reterminal-e1001 …` / `reterminal-e1002` | `main/epaper_status.c`, `main/epd_*.c`, `main/reterminal_sht4x.c` | CH340 `1a86:7523` | Upstream |
| reTerminal Sticky | `tools/board.sh reterminal-sticky …` (Link only: `reterminal-sticky-link`) | `components/muse/boards/board_reterminal_sticky.c`, `main/epd_ssd1677.c` | CH343 `1a86:55d3` | Upstream, experimental |
| reSpeaker Lite with XIAO ESP32-S3 | `tools/board.sh seeed-respeaker-lite …` | `main/voice_board_respeaker_lite.c` | `303a:1001` on the XIAO | Upstream, experimental |
| XIAO AMOLED Display (1.75", 2.05", 2.16") | `tools/muse/board.sh … xiao-amoled-175` / `-205` / `-216` | `components/muse/boards/board_seeed_xiao_amoled.c` | `303a:1001` | This repo, experimental, EVT hardware |
| XIAO ESP32-S3 1.47 Touch Display | `tools/muse/board.sh … xiao147` | `components/muse/boards/board_seeed_xiao_s3_147.c` | `303a:1001` | Upstream ([PR #255](https://github.com/facebookincubator/muse-gadget-sdk/pull/255)) |

"Upstream" means the official Muse Gadget SDK carries the board too. The
helper's `…` is `build` or `flash` (`tools/muse/board.sh build xiao147`,
`tools/board.sh sensecap-indicator flash`). A device
is *experimental* while some of its features haven't been checked on hardware;
the Features table and notes in [`esp32/devices/README.md`](esp32/devices/README.md)
say which. Keep that table, this one and the beginner's guide in step when a
board's status changes.

## At the start of a session

1. Run `git status`. Ask about uncommitted work you didn't make before you
   change anything.
2. Work out which board is attached, as
   [Identify the board first](esp32/AGENTS.md#identify-the-board-first)
   describes, and say what told you. Several boards share the chip's own USB
   (`303a:1001`) and its `chip-id`, so for a board not yet running Muse the
   user's word for the model settles it.
3. Use ESP-IDF v6.0.1. `tools/muse/board.sh` sources `$IDF_EXPORT` when it's
   set. Otherwise both helpers use the `idf.py` on `PATH`, or look for one:
   `tools/muse/board.sh` in `~/.espressif/esp-idf-v6.0.1`, `$IDF_PATH` and
   `~/esp/esp-idf*`, `tools/board.sh` in `$IDF_PATH` and `~/esp/esp-idf*`.
   Check the version with `git -C "$IDF_PATH" describe --tags`.
4. Ask for the user's SDK token: `mgst_` and 43 more characters, 48 in all.
   Every build that will run on a device needs it (see
   [Credentials and backups](#credentials-and-backups)).
5. Before the first flash on a device, back up its flash (see below).

## Building apps and demos

A gadget is useful to Muse through the commands it registers: Muse reads each
command's description in `link.register`, decides when to call it, and calls it
with `link.invoke`. Most apps and demos are one of these:

| The user wants | Build it as | Start from |
|---|---|---|
| Muse to read a sensor | A reading in `sensors.read`, refreshed by a background task | `main/sensecap_sensors.c`, `main/reterminal_sht4x.c`, [Adding a command](esp32/AGENTS.md#adding-a-command) |
| Muse to switch, move or trigger something | A new command, e.g. `relay.set`, gated by a Kconfig option | [Adding a command](esp32/AGENTS.md#adding-a-command) |
| Muse to show something | `display.draw_url` (an image URL) or `display.draw_file` (an image on the VM), or a new command for the screen | `main/noise_control.cpp`, `tools/image_for_display.py` |
| A different look or behaviour on screen | Changes in `components/muse/` (UI, settings) or a new avatar | `muse_ui.c`, `muse_settings_ui.c`, `eink/` for FreeInk boards, [Update my avatar](esp32/AGENTS.md#update-my-avatar), `esp32/simulator/` |
| Muse to control devices already on the network | A skill: Markdown, no firmware change | [`skills/README.md`](skills/README.md) |
| The same on a Raspberry Pi | The Linux SDK | [`linux/AGENTS.md`](linux/AGENTS.md) |

Where the code goes:

- A feature that needs particular hardware gets its own Kconfig option
  (`main/Kconfig.projbuild` or `components/muse/Kconfig`), off unless the
  board's overlay turns it on, with both the advertising and the handling
  wrapped in the same `#if`.
- The board's own pins and chips belong in its board file; shared code reads
  them through `muse_board_t`, so it stays the same on every board. A pin for
  hardware the user adds is a Kconfig value next to its feature's option, both
  off by default and set in that user's build (`idf.py -B <dir> menuconfig`),
  so the shared overlay stays as the board ships. Check the pin against the
  board file's pin defines and the device guide's pin table first: most of a
  small board's pins already carry the screen, touch and microphone.
- Commands run on the session's task: slow work goes on a task of its own and
  answers with `noise_ctrl_send_command_result()`, as `camera.capture` does.
- Every new command or parser gets a host test in `esp32/tests/`.

The loop for trying a change, run from `esp32/`:

```sh
tools/muse/board.sh build xiao147          # or tools/board.sh <board> build
# first build only: put the SDK token in build-muse-seeed-xiao-s3-147/sdkconfig, then build again
tools/muse/board.sh flash xiao147
python3 tools/muse/monitor.py "$(tools/muse/ports.py xiao147)" 30   # boot log
python3 tools/muse/chat.py --status        # the board's state, as JSON
python3 tools/muse/chat.py "What can you do with this gadget?"
```

`chat.py` sends a typed turn to the user's Muse through the board, which is
the quickest way to see whether Muse finds and uses a new command. On a UI
board, `MUSE_BENCH=1` builds take screenshots with `tools/muse/snap.py`.

## Upload to CodeCraft Hub

[CodeCraft Hub](https://sensecraft.seeed.cc/ai/application/) is the community platform where people can discover your project, and easily deploy your Muse Gadget application with one-click.

Uploading is **optional** for general development and use, but **required if
you want to participate in the [Seeed Studio Muse Gadget Challenge](https://test-codecraft.seeed.cc/hub/contest/muse-gadget-challenge).**

Use [`sensecraft-cli`](https://github.com/Seeed-Solution/sensecraft-cli) to upload projects to CodeCraft Hub. Follow the [set up guide](https://test-sensecraft.seeed.cc/wiki/en/sensecraft-cli/guides/getting-started/setting-up/) to install, complete authorization, and load the skills. Then use the /create-app skill to turn this project into a deployable application on CodeCraft Hub.

## Credentials and backups

- The SDK token goes in the build directory's `sdkconfig` as
  `CONFIG_GADGET_SDK_TOKEN="mgst_…"`: `build-<board>/sdkconfig` for
  `tools/board.sh`, `build-muse-<profile>/sdkconfig` for `tools/muse/board.sh`.
  The first build creates that file; set the token there (or with
  `idf.py -B <dir> menuconfig`) and build again. A device that's already
  flashed takes it over serial as `>sdk_token=mgst_…`. Build directories are
  ignored by git. When you log or report the token, show `mgst_` and the first
  few characters only.
- Other keys a demo needs go to the device over USB into NVS, as the token
  does, so neither firmware images nor the repository carry them.
- Back up a device before its first flash, into the repository root's
  `backups/`, which git ignores. From the root:
  `python -m esptool -p PORT read-flash 0 ALL backups/<alias>-<mac>-factory.bin`
  (from `esp32/`, write to `../backups/`).
  The SenseCAP Watcher also needs its `nvsfactory` partition saved, through
  `tools/muse/paced_esptool.py` (see [`esp32/devices/README.md`](esp32/devices/README.md)).
- `idf.py erase-flash` wipes pairing and Wi-Fi as well. Use it when the user
  wants a fresh start, or when a reflashed board still carries another
  firmware's setup in NVS.
- Builds are signed with the development key and keep Secure Boot, flash
  encryption and eFuse pairing off, so a board can be reflashed any number of
  times.

## Hardware scope

Support a board as Seeed ships it: drive the peripherals on the board itself.
A board without a speaker runs as a microphone-only board through the
`mic_init`, `mic_power` and `mic_read` fields of `muse_board_t`
(`components/muse/muse_board.h`), as the reTerminal Sticky and XIAO 1.47 do.
Support for hardware the user adds is a separate feature, behind its own
Kconfig option and in its own PR.

Enable an optional peripheral service by default when a shipped feature uses
it. Otherwise expose it through an opt-in Kconfig option, as the Sticky's
microSD mount is, and verify both settings. Use event waits for idle tasks
where the hardware supports them, and justify any periodic polling the port
adds. Slow screens still present changes within a bounded time while the
user keeps typing or touching the screen.

This repository takes device support, Muse features and fixes. Integrations
with third-party cloud services, and helpers for one person's machine or
network, belong in the user's own fork.

Write down what was checked on hardware and what wasn't: the device's
Features column says `Enabled` for what's built but not yet tried, and the
notes under the table list what was checked, on how many units.

## Changing code and docs

- Follow [`esp32/AGENTS.md`](esp32/AGENTS.md): match the surrounding code's
  style, say Muse (never Hatch) in anything a person reads, and keep
  third-party files' licenses and headers.
- A new board follows steps 1 to 8 in
  [`esp32/devices/AGENTS.md`](esp32/devices/AGENTS.md). In this repository it
  also goes in the [Seeed devices](#seeed-devices) table above, in the
  beginner's guide (`README.md` and `README.zh-CN.md`: supported devices, the
  build, settings folder and confirm button tables), in the CI build matrix
  (`.github/workflows/esp32.yml`), and in `web-flasher/boards.json` with the
  matrix's board name and its console's USB ID, as
  [`web-flasher/README.md`](web-flasher/README.md) describes.
- Seeed's own docs come in English and Chinese: the root READMEs,
  `esp32/README.zh-CN.md` and the device guides (`*.zh-CN.md`). Change both.
- Docs say what to do and how things are; they describe the current state.

## Verifying a change

Before you hand work back:

1. Build each board you touched from a fresh build directory, and one board
   you didn't if shared code changed. Check the overlay reached the config and
   the app-size line, as [Verify](esp32/devices/AGENTS.md#8-verify) describes.
2. Run the host tests from `esp32/`. `tools/muse/board.sh` clears
   `managed_components/`, so run one `idf.py build` first (the default C5
   build is enough), then `python3 -m unittest discover -s tests -p 'test_*.py'`.
   E-ink navigation tests use the pinned FreeInk checkout through
   `FREEINK_SOURCE_DIR` or `managed_components/freeink-sdk` from a Sticky
   configure. Report skipped tests and their missing inputs.
3. For shared LVGL layout or input changes, run the
   [UI simulator tests](esp32/simulator/README.md#automated-tests). Cover the
   new screen and representative existing geometries affected by the change
   (narrow, wide or round), with interaction checks such as text entry, save,
   cancel and return where those flows change. Keep the simulator's Linux
   and macOS CI jobs passing. FreeInk interactions use the e-ink host tests
   and the board.
4. On hardware, exercise the complete path the feature exposes: setup and
   reconnect, Wi-Fi text entry, talk controls and reply display, image format
   and grayscale or colour, sleep/wake, and both logs and commands through
   the console USB port, as applicable. Record build-only results separately.
5. Report the tested commit, board revision and unit count, toolchain,
   commands, build sizes and results, and the checks still awaiting hardware.

## Contributing

### Where a change goes

| Change | Pull request to |
|---|---|
| Seeed device support, Seeed docs, the beginner's guide | This repository, [Seeed-Projects/seeedstudio-muse-gadget-sdk](https://github.com/Seeed-Projects/seeedstudio-muse-gadget-sdk), against `main` |
| A board-neutral fix or feature, or a board ready for the official SDK | Upstream, [facebookincubator/muse-gadget-sdk](https://github.com/facebookincubator/muse-gadget-sdk), against `main` |

A Seeed board lands here first. It's ready for upstream once it's on sale (its
row in [`esp32/devices/README.md`](esp32/devices/README.md)'s Supported
devices table has a store link), it's in the CI build matrix, and its PR says
which features were checked on hardware and which weren't. Upstream takes an
experimental board this way, as it did the Sticky.

Upstream needs Meta's [Contributor License Agreement](https://code.facebook.com/cla)
from every author. The author is the person whose change it is, including
when a coding agent wrote it for them: they sign the CLA and the PR goes up
under their name. An upstream PR carries only what upstream has: no Seeed
beginner's guide, no Chinese docs.

### A pull request

- One change per PR: a board port, a feature, or a fix. A port with an extra
  feature becomes two PRs.
- Branch from the destination repository's current `main` with a `feat/`,
  `fix/`, `docs/` or `refactor/` prefix. For upstream, start from
  `upstream/main` and apply the feature's changes to its current shared files.
- Commit messages: a short summary line, then what changed and why it works.
  A contributor commits and opens the PR under their own name.
- Leave out credentials, firmware images, flash backups, logs with tokens in
  them, and personal network or provisioning files.
- The description has a **Summary** (what changes, file by file for a port),
  **Testing** (each thing checked on hardware, on how many units, then builds
  and host or simulator tests, with the tested commit) and the limits that
  remain. A board PR names the vendor sources its pins come from.
- CI builds the boards listed in `.github/workflows/esp32.yml`; a new board
  adds one entry per profile there. Each web-flasher board id matches one
  build entry; keep existing board ids and feature-table columns intact.

### Reviewing a pull request

Check, in order: the scope (one change, the board as shipped); no credentials;
shared-code changes are gated or provably limited to the new board; the docs
tables still have the same number of columns in every row and the other boards'
cells unchanged; the PR is based on the current `main` (rebase or merge it, and
apply only its diff to shared files); then follow
[Verifying a change](#verifying-a-change), including the shared UI tests and
the complete device interactions affected by the patch. Review default
peripheral activity, idle behaviour and the advertised device capabilities
alongside the driver itself. When only part of a PR fits, ask its author to
split it.
If you take the part that fits yourself, commit it under your own name with
the PR's number in the summary line (`(#12)`), comment on the PR with what
went in and what didn't, and close it; the author can send the rest as their
own PRs.

### After review and merge

Reviewers may adjust the implementation or add regression coverage. Verify
the final reviewed revision: rerun the affected automated checks, and repeat
affected hardware flows when behaviour changed since the last device test.
The contributor with the device supplies those results, or records which
checks await hardware. Record that revision and its results in the PR;
maintainers' build and host test results complement the contributor's
hardware results. After merge, check the landed commit and refresh device
status and validation notes from that implementation.

## Keeping up with upstream

```sh
git remote add upstream https://github.com/facebookincubator/muse-gadget-sdk.git   # once
git fetch upstream
git merge --no-ff upstream/main
```

For a device accepted upstream, compare the submitted patch with the landed
commit and retain the reviewed implementation. Keep this repository's
remaining additions beside it, such as the AMOLED profiles and Chinese docs.
Reconcile board lists, component rules, CI and web-flasher entries by board id
so each profile occurs once, and preserve the existing feature-table cells.
Then build the Seeed boards, run the affected host and simulator tests, and
bring the Chinese docs up to date with the English changes upstream made.
