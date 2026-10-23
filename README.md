# Demul Guest Debugger (SH-4 / Dreamcast & Arcade)

A native, in-process SH-4 / SH-3 debugger, disassembler and reverse-engineering toolkit for the **Demul v0.7 (Build 251220)** emulator.

By **LuC-iTA @ Gamehacking.org**

<img width="1280" height="694" alt="demul_debugger" src="https://github.com/user-attachments/assets/d6fb012a-5839-4442-98b2-ff215b2d13f5" />

---

## What it is

The whole tool is a single file, **`padDemulDbg.dll`**. It is a complete Demul pad (controller) plugin that also hosts the debugger. Demul loads it through its own plugin loader, so there is no launcher and no injection.

> [!IMPORTANT]
> **Upgrading from an older release?** The former `demul_debugger.exe` launcher and `debugger_core.dll` are obsolete. Delete both from your Demul folder and install `padDemulDbg.dll` as described below.

---

## Requirements

- **Windows**.
- **Demul v0.7 Build 251220, exactly.** The plugin checks the host executable at start-up. On any other build it works only as a pad plugin and installs no debugger hooks.
- **Interpreter mode** for the debugger: `cpumode = 0` in `Demul.ini`. The debugger does not work with the dynarec.

---

## Installation

1. Download `padDemulDbg.dll` from the [Releases](https://github.com/LuC-iTA-GHorg/demul_debugger/releases) page.
2. Copy it into Demul's `plugins\` folder.
3. Select it as the pad plugin, either:
   - in Demul: **Config → Plugins and Paths → Controls → "Demul Debugger Pad (padDemulDbg)"**, or
   - in `Demul.ini`, section `[plugins]`: `pad = padDemulDbg.dll`
4. Set `cpumode = 0` in `Demul.ini`.
5. Configure your buttons as usual via **Config → Controls**. The plugin has its own configuration dialogs and uses the same `padDemul.ini` as the stock pad plugin, so your existing mappings carry over. The stock `padDemul.dll` is not needed (it can stay in the folder as an alternative).

---

## Usage

Start Demul and launch a game as you normally would, from the menus or from the command line, for example:

```pwsh
.\demul.exe -run=cave3rd -rom=ibara
.\demul.exe -run=alien -rom=gekimaka
```

Open the debugger windows from the **Debug Tools** menu or with the hotkeys below.

| Category | Shortcut | Function |
| :--- | :--- | :--- |
| **Window Toggles** | **Ctrl + D** | Show / Hide Main Disassembler Window |
| | **Ctrl + W** | Open Branch Watcher & Dynamic Flow Profiler |
| | **Ctrl + M** | Open Real-Time Memory Viewer & Hex Editor |
| | **Ctrl + K** | Open Dedicated Stack Viewer |
| | **Ctrl + S** | Open Real-Time Memory Scanner |
| | **Ctrl + B** | Open Breakpoint, Watchpoint & Memory Freeze Manager |
| | **Ctrl + T** | Open Instruction Trace Logger |
| | **Ctrl + H** | Open SH-4 Converter Scratchpad (ASM ↔ HEX) |
| **Execution Controls** | **Ctrl + G** | Jump Disassembler to Guest Address |
| | **F2** | Toggle Execution Breakpoint on selected instruction / address |
| | **F5** | Pause / Resume Emulation (toggle) |
| | **F9** | Resume Emulation |
| | **F7** | Step Into (Single SH-4 Opcode) |
| | **F8** | Step Over (Subroutine Call) |
| | **F6** | Step Out (Return from Subroutine) |
| **Window Actions & Navigation** | **Space** | Toggle Recording in Trace Logger or Branch Watcher |
| | **Up / Down** | Line-by-line selection & scrolling in Disassembler, Memory Viewer & Trace Logger |
| | **PageUp / PageDown** | Page-by-page scrolling in Disassembler & Trace Logger |
| | **Home / End** | Jump to newest / oldest entry in Trace Logger |
| | **Enter / Esc** | Confirm input in address jump boxes & dialogs / Close dialogs |

The window toggles work whenever a Demul window (main window, game window or any debugger window) has the keyboard focus. They are released while another application is in the foreground, so they never steal those key combinations from other programs. `Ctrl + G` and the function keys act on the disassembler window when it has the focus.

---

## Features

- **SH-4 disassembler and assembler**: fully symmetric over the entire opcode space, including FPU and vector instructions; branch arrows, target labels, literal-pool previews and in-place assembly.
- **Breakpoints and stepping**: execution breakpoints, step into / over / out, pause and resume.
- **Hardware watchpoints (DR0–DR3)**: on main RAM and on the BIOS, flash, VRAM and ARAM regions.
- **Instruction trace logger**: ring buffer of recent PC, opcode and key registers.
- **Branch watcher and flow profiler**: records every conditional branch that executes; invert BT/BF, force a jump or NOP a branch in place; export the log.
- **Memory viewer and hex editor**: with tabs for the real BIOS, VRAM, ARAM, flash and ROM regions of the running system.
- **Stack viewer**: frames relative to R15, annotated with return addresses.
- **Memory searcher and freezer**: exact / changed / unchanged / increased / decreased / between scans; lock 1-, 2- and 4-byte and float values.
- **Live patching**: edit guest code and data while the game runs.
- **ASM ↔ HEX scratchpad** with literal-pool resolution.
- **Disc extraction**: extracts and descrambles `1ST_READ.BIN` from mounted GDI / CDI images.

---

## Supported systems

| System | Status |
| :--- | :--- |
| Sega Dreamcast | Verified |
| Sega NAOMI 1 | Verified |
| Sammy Atomiswave | Verified |
| Cave CV1000 | Verified |
| Capcom / Sega Medalusion | Verified |
| Sega Hikaru | Untested |
| Sega NAOMI 2 | Untested |
| Gaelco SH-4 (ATV Track, Gaelco Football, Smashing Drive) | Untested |

Gaelco titles need Demul's `gpuDX11old.dll` GPU plugin, which in turn requires the [DirectX End-User Runtimes (June 2010)](https://www.microsoft.com/en-us/download/details.aspx?id=8109).

---

## Environment switches

Set these before starting Demul:

- `PADDEMULDBG_TRACE=1`: log one line for every press and release of each bound input.
- `PADDEMULDBG_NO_DEBUGGER=1`: pad plugin only; no debugger hooks are installed.

---

## Output files

- `debugger_log.txt`, next to `demul.exe`: plugin and debugger start-up log.
- `dumps\`, in the Demul folder: extracted boot files and RAM dumps.

---

## Known limitations

- The debugger works only with the SH-4 interpreter (`cpumode = 0`), not the dynarec.
- An XInput pad may appear twice in the device list.
- Rumble / force feedback is not supported yet.
- Writes to the BIOS region are refused by design.

---

## Troubleshooting

**Ctrl+D does nothing.** Check, in order:

1. "Demul Debugger Pad (padDemulDbg)" is selected under Config → Plugins and Paths → Controls.
2. You are running Demul v0.7 **Build 251220**. On any other build the debugger is not started.
3. `cpumode = 0` is set in `Demul.ini`.
4. `debugger_log.txt` next to `demul.exe` reports what happened during start-up.

---

## Antivirus / SmartScreen

The current `padDemulDbg.dll` was not flagged in the author's check on 23 October 2026 (the old launcher-based release was). If an antivirus engine does flag it, that is a heuristic reacting to in-process hooking, which a native debugger cannot avoid; please report it to the vendor as a false positive.

**Only download the DLL from this repository's Releases page. Never repackage, pack, compress or rename it**: that changes nothing about what it does and makes detections more likely.

---

## Disclaimer

This project is not affiliated with or endorsed by the authors of Demul. Use it only with games and BIOS files you legally own.
