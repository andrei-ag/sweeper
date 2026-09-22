# 🩺 the Sweeper 012/beta

[![License: GPL v3](https://img.shields.io/badge/License-GPLv3-blue.svg)](https://www.gnu.org/licenses/gpl-3.0)
[![Language](https://img.shields.io/badge/ASM-99.9%25-orange)](https://github.com/andrei-ag)
[![Version](https://img.shields.io/badge/version-012BETA-red)](https://github.com/andrei-ag)

**Educational DOS antivirus demonstrating heuristic decryptor analysis, narrow x86 emulation, and MBR/BOOT disinfection.**

> ⚠️ **Important:** This software is for **educational and research purposes only**. It is a historical DOS-era antivirus written in 2000. It should not be used on production systems. Run it only in isolated environments (e.g., DOSBox, a virtual machine) with backup copies of files. Some routines write directly to disk sectors and modify executable files.

## 🎯 Key Features

### 🧠 Narrow x86 Emulator (Assembly)
- Implements a narrow x86 instruction emulator written from scratch in assembly (~10 opcodes: `INC`, `RET`, `JZ`, `LOOP`, `MOV`, `CMP`, `XOR`, …).
- Designed specifically to execute setup code of DOS polymorphic decryptors without running the payload.
- Sandboxes instruction execution: register state is held in `word ptr [bp+reg_??]`, memory writes go to the in-memory file buffer, not to disk.
- Handles `JZ` and `LOOP` with correct sign-extension for backward jumps.
- Special handling of `XOR WORD PTR [SI], IMM` — the classic decryption primitive — by resolving the file offset and applying the XOR in the sandbox.

### 🔍 Heuristic Polymorphic Decryptor Analysis
- Junk filter — drops one-byte noise (`NOP`, `CLI`, `CLC`, segment prefixes, `PUSH`/`POP` all registers).
- Decryptor builder — scans the entry point region for `LEA`, `CMP`, short `JMP`, `JNE`, `LOOP`; follows jumps; reconstructs the decryptor.
- JNE-cycle analysis — detects the classic `CMP reg, imm16` / `JNE` pair, self-modifies the `CMP` into a `MOV`, and *emulates* the setup to read the counter — i.e., the encrypted payload size.
- Loop analysis — temporarily patches the loop instruction with `RET`, calls the decryptor in the sandbox, and captures the resulting `CX` as the payload size.

### 🩺 Virus Disinfection
The antivirus can not only detect but also remove virus code and restore infected files. Disinfection routines are implemented for:
- ✅ **One_Half.3544** (COM and EXE) — restores 10 "spots" from the footer
- ✅ **NYB** (BOOT) — rewrites 14 sectors of track 0, head 1
- ✅ Pieck.4444 (EXE) — rebuilds the MZ header from a saved copy
- ✅ Yankee_Doodle.2885 (COM and EXE) — shifts the body back by 1991 bytes
- ✅ VBS/LoveLetter, VBS/FreeLinks, mIRC/Jeepwarz — deleted by signature

### 🔍 DOS File & Boot Sector Analysis
- Analysis of MZ/COM structure, COM entry via `JMP` opcode, EXE entry via header fields.
- CRC16/CRC32 computation for signature matching (`calculate_crc`).
- Boot sector and MBR access via `INT 13h` for NYB detection.
- VBS / BAT / INI analysis via CRC16 of individual lines — defense against polymorphic comment insertion.

### 🖥️ Console Interface (Russian / English)
- Command-line: `SWEEPER {PATH}`.
- Interactive Setup (`SETUP.ASM`) — writes `SWEEPER.CFG` with cure/scan mode, deep scan flag, and report options.
- Localization: english, russian.
- Scan report appended to a log file (see `SWEEPER.LOG` for a real 2000 sample).

### 🔑 Registration Keygen
- `SWPR_KGN.ASM` — a standalone keygen producing `SWEEPER.KEY`, demonstrating the era's copy-protection approach (CRC16 of name/org/email + random block).

## 🔗 Lineage: from the Sweeper to Explosion Antivirus

the Sweeper (2000) and [Explosion Antivirus](https://github.com/andrei-ag/xpl_av) (2004–2009) were written by me four years apart, and they trace a clean line of evolution:

| Concept | the Sweeper (2000, DOS) | Explosion AV (2004–2009, Win32) |
| :--- | :--- | :--- |
| **Platform** | DOS, real mode, `.model tiny` | Windows, protected mode, PE |
| **Targets** | COM, EXE, BAT, VBS, INI, MBR, BOOT | Win32 PE, DLL (`KERNEL32.DLL`) |
| **Emulator** | Narrow, 27 opcodes | 104 opcode slots (~56 handlers, 1000+ forms), FPU, 31 Win32 API |
| **Mutant detection** | One_Half.3544 — via full decryptor pipeline (heuristic → emulation → decrypt → signature) | Win32/Driller — ~9 KB polymorphic decryptor with anti-emulation API calls |
| **Anti-emulation** | Not addressed | Dedicated topic (article "Vulnerabilities of Code Emulators", 2004) |
| **Loop protection** | `cmp cx, 200` | Loop detector |
| **Unpacking** | One layer, manual | Universal, until all layers are stripped |
| **PE analysis** | Only MZ header | Full PE: sections, imports, exports |
| **Disinfection** | Overwrite spots at fixed offsets | Restore PE structure (Parite, Krized, Funlove, Marburg) |
| **Assembler** | TASM + TLINK | Flat Assembler (FASM) |
| **License** | Copyright, re-released 2026 | GNU GPL v3 |

## 🚀 Getting Started

### Build Requirements
- Borland Turbo Assembler (TASM) 4.0 or compatible.
- Borland Turbo Link (TLINK).
- A DOS environment (real or emulated).

See `SCANER.BAT`, `MAKE_ENG.BAT`, `MAKE_RUS.BAT`, `MAKE_VBA.BAT`, `CRYPT.BAT`, `CRC.BAT`, `SWPR_KGN.BAT` for the original build scripts.

## 📂 Repository Structure

| Directory/File | Description |
| :--- | :--- |
| `SCANER.ASM` | Main source file — scanner, loader, UI |
| `BASE.INC` | Virus base work library |
| `KERNEL.INC` | Kernel dispatcher (16 functions) for the external virus base |
| `SETUP.ASM` | Interactive configuration writer (`SWEEPER.CFG`) |
| `SWEEPER.LOG` | Sample scan report from 2000 |
| `BASE\COUNT.INC` | Mini-disassembler ("Small disasm", (c) Reminder 1997) |
| `BASE\CRYPT*.ASM` | Encryption utilities for language and base files |
| `BASE\CUREPROC.INC` | Virus disinfection routines |
| `BASE\DETPROC.INC` | Virus detection routines |
| `BASE\PROCS.INC` | Common procedures (file, time, console, CRC) |
| `BASE\SIGS.INC` | Signatures (byte arrays and CRC16 line tables) |
| `BASE\VIR_BASE.ASM` | External virus base source (assembles into `SWEEPER.DAT`) |
| `BUILDER\BUILDER.INC` | Heuristic decryptor builder |
| `BUILDER\EMULATE.INC` | Narrow x86 emulator |
| `BUILDER\FUNC.INC` | Helper routines for loop / instruction analysis |
| `BUILDER\JNE_ANL.INC` | JNE-cycle decryptor analysis |
| `BUILDER\POLYMORP.INC` | One-byte junk filter and garbage table |
| `LANGUAGE\ENGLISH.ASM` | English language file source |
| `LANGUAGE\RUSSIAN.ASM` | Russian language file source |
| `SWPR_DOC\HISTORY.TXT` | Original changelog (Russian) |
| `SWPR_DOC\SWEEPER.DOC` | Original user manual (Russian) |
| `SWPR_KEY\SWPR_KGN.ASM` | Registration key generator |
| `SWPR_UTL\CRC\*` | CRC16/CRC32 test utilities |

### Virus Base

`SWEEPER.DAT` is the encrypted virus base. It is loaded into memory, decrypted with the same rolling-XOR as the language file, and dispatched through a small kernel (`KERNEL.INC`) by function number. Updating the base does **not** require rebuilding the scanner.

## 🚀 Usage

```
SWEEPER { PATH }
```

### Setup

Run `SETUP.EXE` once. It asks:

- **Language** — `R` for Russian, `E` for English.
- **Scan or cure** — `S` for scan only, `C` for scan and cure.
- **Deep scan** — `Y` for heuristic analysis of decryptors, `N` for signatures only.
- **Report** — `Y` / `N`, then `A` (append) / `R` (recreate), then the report file name.

The result is written to `SWEEPER.CFG`.

### Examples

Scan drive C:
```
SWEEPER C:\
```

Scan a directory:
```
SWEEPER C:\SAMPLES
```

Press **ESC** during a scan to stop.

## 📄 License

This project is re-released under the **GNU General Public License v3**. A copy of the license is included in `LICENSE.txt`.

> **Note on third-party files:** `COUNT.INC` includes a mini-disassembler originally published by **Reminder** in 1997

## 🙏 Acknowledgements & Historical Note

The original version of this antivirus dates back to **2000** and was written by me under the banner **C. Thomas Hawell Computing** [CTHC]. Development ran roughly from March to August 2000; version `012/beta` is dated 24 August 2000 (see `HISTORY.TXT`).

The virus base reflects the era: DOS file infectors (One_Half, TMC, Yankee_Doodle), boot viruses (NYB), the first generation of VBS worms (LoveLetter), and the earliest PE worms (PrettyPark, SKA). Most of these have been extinct for two decades; they are preserved here as documentation of a vanished ecosystem.

Special thanks to the **DOS scene** of the late 1990s and early 2000s — for the shared knowledge, the newsletters, the BBS files, and the spirit of building things from scratch.

Source codes and releases were published on https://www.sac.sk (use search text 'the Sweeper').

© 2000 C. Thomas Hawell Computing [CTHC]. Re-released for preservation, 2026.
