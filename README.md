# Demul Guest Debugger (SH-4 / Dreamcast & Arcade)

Native, high-performance in-process guest debugger, disassembler, profiler, and reverse engineering toolkit for the **Demul v0.7 (Build 251220)** emulator, written in **Zig** and Win32.

> [!WARNING]
> **Demul Version Compatibility**: This debugger was developed and verified **EXCLUSIVELY for Demul v0.7 (Build 251220 / `demul_251220`)**. Any other build or version of Demul is untested and may crash or fail to function due to static memory offset targets and hook addresses.

> [!CAUTION]
> **Antivirus / SmartScreen False Positives**: The compiled binaries in this repository are very likely to be flagged by Windows Defender, other antivirus engines, or VirusTotal as a trojan, "game hacking tool", or generic malware. **This is a false positive**, but an expected one — read [why this happens and what to do about it](#antivirus--smartscreen-false-positives) before reporting it as a bug or assuming the repository has been compromised.

---

## Supported Systems & Platforms

- **Sega Dreamcast** (16 MB Main RAM)
- **Sammy Atomiswave** (16 MB Main RAM)
- **Sega NAOMI 1 & NAOMI 2** (32 MB Main RAM)
- **Sega Hikaru** (32 MB Main RAM)
- **Gaelco 3D** (8 MB Main RAM)
- **Cave CV1000** (16 MB Main RAM)
- **Capcom / Sega Medalusion** (16 MB Main RAM)

---

## Key Features

- **SH-4 Disassembler & Assembler**: Complete, fully symmetric Hitachi SH-4 encoding and decoding - every instruction the disassembler prints is accepted back by the assembler (zero unsupported encodings across the whole 65,536-opcode space), including `fmov.s` indirect forms, `fmac`, the GBR-relative byte logicals, banked registers and the vector FPU instructions. Verified opcode-by-opcode across the full 65,536-encoding space by round-trip and by differential comparison against an independent reference decoder (see `BUG_AUDIT_REPORT.md`), with branch visual arrows, target labels (`; loc_...`, `; sub_...`), literal pool preview (`; = 0x...`), Gutter markers, and in-place assembly.
- **Branch Watcher & Flow Profiler (`Ctrl+W`)**: Real-time branch execution profiler supporting 8,192 unique branch sites and 16,384 hash slots. Displays preceding condition instruction (`PC - 2`), supports "Cond Only" filtering, idle noise calibration, event isolation, in-place condition inversion (`BT` $\leftrightarrow$ `BF`), force jump, NOP, and log export.
- **Single-Cycle Bitwise Opcode Hook**: Matches conditional branches in 1 CPU cycle inside the interpreter step hook (`0x00511EB0`).
- **Instruction Trace Logger (`Ctrl+T`)**: 2048-entry circular ring buffer recording PC, opcode, PR, R0, R15, and SR with zero heap allocations.
- **In-Process Resilient RAM Writing**: Directly modifies guest code and variables via `VirtualProtect(PAGE_EXECUTE_READWRITE)`, preventing write failures in Win32 WOW64 environments.
- **Hardware Memory Watchpoints (`DR0`..`DR3`)**: Zero-latency hardware watchpoints powered by x86 CPU debug registers and Vectored Exception Handling. Addresses are masked against the live RAM size (8 / 16 / 32 MB), and misaligned watchpoints are rejected up front because the debug registers would never fire for them.
- **Memory Freeze / Value Locker**: Real-time thread-safe memory lock engine for 1B, 2B, 4B, and 32-bit Float values running on a 1ms monitor loop.
- **RAM Search Engine (`Ctrl+S`)**: Multi-type memory scanner (Exact, Changed, Unchanged, Increased, Decreased, Between) with alignment filtering.
- **Memory Viewer & Hex Editor (`Ctrl+M`)**: Dynamic full-viewport hex dump with instant jumps to RAM, VRAM, ARAM, and BIOS.
- **Dedicated Stack Viewer (`Ctrl+K`)**: Real-time call stack and frame inspector relative to R15 with return address annotations.
- **SH-4 Converter Scratchpad (`Ctrl+H`)**: Two-panel bidirectional ASM $\leftrightarrow$ HEX converter with automatic literal pool resolution.
- **Direct Disc Extraction (`1ST_READ.BIN`)**: Traverses mounted GDI/CDI images and ISO9660 directory structures, with automatic 2MB tile Sega descrambling into `dumps/1ST_READ_DESCRAMBLED.BIN`.
- **Dynamic UI Control Lock & Cold-Start Alignment**: Tracks Demul emulation lifecycle via `GetExitCodeThread`. Displays *"Emulation Inactive"* banners across all windows on cold start, clearing automatically via live timer invalidation on game start. Detects Dynarec vs. Interpreter (`0x0A4859C8`) on-the-fly across all platforms, locking execution hooks while keeping Memory Freeze active.
- **Fully Relocatable Installation**: No absolute path is compiled into the binaries. Simply place `demul_debugger.exe` and `debugger_core.dll` in the same folder as `demul.exe` - any folder, any drive, including removable media. The launcher refuses to start if `demul.exe` is not adjacent, and still validates that it is exactly Demul v0.7 Build 251220 before injecting.
- **Audited Correctness, Zero Known Defects**: The September 2026 source audit found and closed 17 defects. Across the full 65,536-opcode space the disassembler and assembler are exactly symmetric - 0 unsupported encodings, 0 round-trip divergences - and the decoder matches an independent reference with zero false acceptances. Every finding carries a dedicated regression test.
- **Minimal Binary Footprint**: Core DLL is optimized at **~409 KB** and Launcher at **~397 KB** (strictly under the 600 KB limit) through `.bss` buffer paging and `-O ReleaseSmall -fstrip`.

---

## Shortcuts & Controls

| Category | Shortcut / Button | Function |
| :--- | :--- | :--- |
| **Window Toggles** | **Ctrl + D** | Show / Hide Main Disassembler Window |
| | **Ctrl + W** | Open Branch Watcher & Dynamic Flow Profiler |
| | **Ctrl + M** | Open Real-Time Memory Viewer & Hex Editor |
| | **Ctrl + K** | Open Dedicated Stack Viewer |
| | **Ctrl + S** | Open Real-Time Memory Scanner |
| | **Ctrl + B** | Open Breakpoint, Watchpoint & Memory Freeze Manager |
| | **Ctrl + T** | Open Instruction Trace Logger |
| | **Ctrl + H** | Open SH-4 Converter Scratchpad (ASM $\leftrightarrow$ HEX) |
| **Execution Controls** | **Ctrl + G** | Jump Disassembler to Guest Address |
| | **F2** | Toggle Execution Breakpoint on selected instruction / address |
| | **F5** | Pause Emulation |
| | **F9** | Resume Emulation |
| | **F7** | Step Into (Single SH-4 Opcode) |
| | **F8** | Step Over (Subroutine Call) |
| | **F6** | Step Out (Return from Subroutine) |
| **Window Actions & Navigation** | **Space** | Toggle Recording in Trace Logger or Branch Watcher |
| | **Up / Down** | Line-by-line selection & scrolling in Disassembler, Memory Viewer & Trace Logger |
| | **PageUp / PageDown** | Page-by-page scrolling in Disassembler & Trace Logger |
| | **Home / End** | Jump to newest / oldest entry in Trace Logger |
| | **Enter / Esc** | Confirm input in address jump boxes & dialogs / Close dialogs |

---

## Antivirus / SmartScreen False Positives

If Windows Defender, another antivirus, or VirusTotal flags `demul_debugger.exe` or
`debugger_core.dll` as a trojan, "hack tool", "riskware", or similar — **this is expected, and it is
a false positive.** No malicious code has ever been present in this repository. This section
explains exactly why the detection happens, what actually reduces it, and what does not.

### Why antivirus engines flag this project

Antivirus heuristics do not read intent, they read **behaviour**. And the behaviour this project
needs in order to work is, byte for byte, the same behaviour a game trainer, a cheat engine, or a
process injector uses:

- `demul_debugger.exe` calls `VirtualAllocEx` + `WriteProcessMemory` + `CreateRemoteThread` on
  `LoadLibraryA` to load `debugger_core.dll` into Demul's process. This exact API sequence is the
  textbook signature of DLL injection, and it is what every injection-detection heuristic looks for.
- `debugger_core.dll`, once loaded, installs inline hooks on Demul's interpreter dispatch loops via
  `VirtualProtect(PAGE_EXECUTE_READWRITE)`, patches live instruction bytes in the guest's emulated
  RAM, and registers a Vectored Exception Handler.
- It also programs the x86 hardware debug registers (`DR0..DR3`) directly — the same mechanism used
  by hardware breakpoints in a debugger and by anti-debug bypass tools alike.

Every one of these is necessary for a native, zero-overhead, in-process SH-4 debugger to exist at
all — there is no way to hook an interpreter loop, patch live game code, or set a hardware
watchpoint without doing exactly these things. The heuristics are not wrong about what the code
does; they simply cannot distinguish "debugger for a game emulator" from "cheat injector for a
game" or "trojan targeting a game process", because from the outside those three tools are
indistinguishable at the API-call level. **A 100% clean scan on every engine is not a realistic
goal** for a tool built this way, on any codebase, in any language.
