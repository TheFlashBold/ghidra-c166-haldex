# Ghidra C166 Haldex

Experimental Ghidra language module for Infineon C166/XC166 firmware built with
the Tasking VX-C166 toolchain (e.g. Haldex Gen 5 controllers).

- separate language id: `C166HALDEX:LE:16:default` (compiler spec `tasking`)
- removes compiler-spec references to registers missing from the compiled SLEIGH (`CP`, `MDH`, `MDL`, `MDC`)
- models the Tasking stack as `r15`, matching prologues such as `mov [-r15],...`
- replaces the old placeholder `push`/`pop` userops with real r15-backed stack p-code
- keeps CALL/RET return-address bookkeeping out of the decompiler-visible stack to avoid noisy synthetic return-address stores
- uses a loose Tasking register prototype so helper-heavy code keeps visible register inputs (`r2`..`r5`, `r8`..`r12`)

## Install

Copy this folder to:

```text
~/Library/ghidra/ghidra_12.1.2_PUBLIC/Extensions/ghidra-c166-haldex
```

Restart Ghidra, then load the firmware as raw binary with language
`C166HALDEX:LE:16:default`.

## v1.1 — XC166 / C166S V2 completion

- **MAC unit (CoXXX)**: all 176 CoXXX encodings (opcodes 83/93/A3/B3/C3/D3) from the
  C166S V2 Architecture Overview: CoMUL*/CoMAC*/CoLOAD*/CoADD/CoSUB/CoSHL/CoSHR/CoASHR/
  CoCMP/CoMIN/CoMAX/CoABS/CoNEG/CoRND/CoNOP/CoSTORE/CoMOV with `[Rw±]`, `[Rw±QRx]`,
  `[IDXi±QXx]` post-modification. Registers MAL/MAH (+ 32-bit `MAC_ACC`), MSW, MCW,
  MRW, IDX0/1, QX0/1, QR0/1. Arithmetic = `MAC_ACC = mac_<mnemonic>(MAC_ACC, op1, op2)`.
- **JMPA/CALLA +/- prediction and loop hint** (`jmpa-`, `jmpa-l`, …); extended USR
  conditions decode as `cc_x<n>_<cc>` via `usr_cond()`.
- **DPP paging** for direct 16-bit `mem` operands via context `DppSet`/`Dpp0Pg..Dpp3Pg`
  (DppSet=0 = legacy identity mapping, so plain C167 imports behave as before).
  The DPP context must be set as a `noflow` program-context value.
- **EXTP/EXTS** (immediate and register form) are applied to direct and indirect
  accesses (previously parsed but ignored).
- XC166 SFR aliases: CPUCON1/2, SPSEG, VECSEG, CPUID.
- `[Rw+#disp]` with disp >= 0x200 (table/struct base as absolute near address) is
  DPP-paged by the displacement's DPP bits → switch/jump tables resolve.
- r15 gets the same direct stack addressing as r0 → far fewer functions with broken
  stack analysis.
- cspec: r8..r10 are callee-saved → no more `extraout_r8` breaking pointer data flow
  across calls.

## Credits

Based on the C166 Ghidra language module by Keyhan Asadi, which served as the
starting point; this module extends and completes it substantially.
