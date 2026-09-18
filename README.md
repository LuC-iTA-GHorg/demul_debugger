# Demul Guest Debugger (SH-4 / Dreamcast & Arcade)

Native, high-performance in-process guest debugger, disassembler, profiler, and reverse engineering toolkit for the **Demul v0.7 (Build 251220)** emulator, written in **Zig** and Win32.

> [!WARNING]
> **Demul Version Compatibility**: This debugger was developed and verified **EXCLUSIVELY for Demul v0.7 (Build 251220 / `demul_251220`)**. Any other build or version of Demul is untested and may crash or fail to function due to static memory offset targets and hook addresses.

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

- **SH-4 Disassembler & Assembler**: Complete Hitachi SH-4 decoding with branch visual arrows, target labels (`; loc_...`, `; sub_...`), literal pool preview (`; = 0x...`), Gutter markers, and in-place assembly.
- **Branch Watcher & Flow Profiler (`Ctrl+W`)**: Real-time branch execution profiler supporting 8,192 unique branch sites and 16,384 hash slots. Displays preceding condition instruction (`PC - 2`), supports "Cond Only" filtering, idle noise calibration, event isolation, in-place condition inversion (`BT` $\leftrightarrow$ `BF`), force jump, NOP, and log export.
- **Single-Cycle Bitwise Opcode Hook**: Matches conditional branches in 1 CPU cycle inside the interpreter step hook (`0x00511EB0`).
- **Instruction Trace Logger (`Ctrl+T`)**: 2048-entry circular ring buffer recording PC, opcode, PR, R0, R15, and SR with zero heap allocations.
- **In-Process Resilient RAM Writing**: Directly modifies guest code and variables via `VirtualProtect(PAGE_EXECUTE_READWRITE)`, preventing write failures in Win32 WOW64 environments.
- **Hardware Memory Watchpoints (`DR0`..`DR3`)**: Zero-latency hardware watchpoints powered by x86 CPU debug registers and Vectored Exception Handling.
- **Memory Freeze / Value Locker**: Real-time thread-safe memory lock engine for 1B, 2B, 4B, and 32-bit Float values running on a 1ms monitor loop.
- **RAM Search Engine (`Ctrl+S`)**: Multi-type memory scanner (Exact, Changed, Unchanged, Increased, Decreased, Between) with alignment filtering.
- **Memory Viewer & Hex Editor (`Ctrl+M`)**: Dynamic full-viewport hex dump with instant jumps to RAM, VRAM, ARAM, and BIOS.
- **Dedicated Stack Viewer (`Ctrl+K`)**: Real-time call stack and frame inspector relative to R15 with return address annotations.
- **SH-4 Converter Scratchpad (`Ctrl+H`)**: Two-panel bidirectional ASM $\leftrightarrow$ HEX converter with automatic literal pool resolution.
- **Direct Disc Extraction (`1ST_READ.BIN`)**: Traverses mounted GDI/CDI images and ISO9660 directory structures, with automatic 2MB tile Sega descrambling into `dumps/1ST_READ_DESCRAMBLED.BIN`.
- **Dynamic UI Control Lock & Cold-Start Alignment**: Tracks Demul emulation lifecycle via `GetExitCodeThread`. Displays *"Emulation Inactive"* banners across all windows on cold start, clearing automatically via live timer invalidation on game start. Detects Dynarec vs. Interpreter (`0x0A4859C8`) on-the-fly across all platforms, locking execution hooks while keeping Memory Freeze active.
- **Minimal Binary Footprint**: Core DLL is optimized at **~406 KB** and Launcher at **~397 KB** (strictly under the 600 KB limit) through `.bss` buffer paging and `-O ReleaseSmall -fstrip`.

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
