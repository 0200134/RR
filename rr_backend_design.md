# ⚙️ RR Backend Design Document  
> RR Compiler Backend Architecture — NASM / ELF / PE  
> Version 0.1 (Internal Draft)

---

## 🧭 1. Purpose
The RR backend transforms the **RR Intermediate Representation (RIR)** into **deterministic machine code**.  
Unlike traditional backends (LLVM, GCC), RR’s backend:
- Emits **human-readable assembly** (NASM format)
- Is **fully deterministic** (1:1 mapping from RIR → ASM)
- Avoids any runtime- or platform-dependent optimization
- Is suitable for **bare-metal, industrial, and LTSS** systems

---

## 🧩 2. Backend Overview




RIR (.rir)
↓
RIR Optimizer (deterministic)
↓
Instruction Selector
↓
Code Generator (NASM / ELF / PE)
↓
Assembler (rra)
↓
Linker (rrl)
↓
Binary Output (.bin / .elf / .exe)



---

## 🧱 3. RIR → ASM Mapping Model

### 3.1 Design Principles
1. **One IR instruction = one ASM block**
2. **No speculative optimization**
3. **Symbol names preserved**
4. **All code emission reproducible (byte-identical)**
5. **Generated ASM must be readable and traceable**

---

### 3.2 Mapping Table

| RIR Instruction | NASM Output | Description |
|-----------------|--------------|--------------|
| `%1 = CONST 5` | `mov rax, 5` | Constant literal |
| `%2 = ADD %1, 3` | `add rax, 3` | Arithmetic op |
| `CALL print %2` | `mov rdi, %2`<br>`call print` | Function call |
| `RET 0` | `mov rax, 0`<br>`ret` | Return value |
| `STORE %a, %b` | `mov [%a], %b` | Memory store |
| `LOAD %x, addr` | `mov %x, [addr]` | Memory load |

---

## 🧰 4. Instruction Selection

The backend uses a **pattern-matching selector**:
- No optimization passes
- Each RIR opcode maps directly to a predefined template

### Example Template (C-like pseudocode)
```cpp
emit_instr(RIR_Opcode op) {
    switch (op) {
        case RIR_ADD: out("add %s, %s", reg(a), reg(b)); break;
        case RIR_SUB: out("sub %s, %s", reg(a), reg(b)); break;
        case RIR_MOV: out("mov %s, %s", reg(a), reg(b)); break;
        case RIR_CALL: out("call %s", fn_name); break;
        case RIR_RET: out("ret"); break;
    }
}



Templates are version-frozen once released to preserve reproducibility.



🧩 5. NASM Emitter


5.1 Output Format


section .text
global main
main:
    mov rax, 5
    mov rbx, 7
    add rax, rbx
    call print
    mov rax, 0
    ret

section .data
msg db "Hello, Industry!", 0



5.2 Emission Phases




Header Generator — inserts metadata, target info


Symbol Resolver — defines function and data labels


Instruction Writer — emits deterministic opcode blocks


Section Builder — splits .text / .data / .bss





🧩 6. Assembler (rra)


RR provides its own minimal assembler:




Supports subset of NASM syntax (strict deterministic mode)


Converts .asm → .o object


No optimization, no randomization


Outputs symbol table (.sym) for verification




Example


rra main.asm -o main.o



Output:


Symbol Table:
  _start
  main
  print
Sections:
  .text (size: 84)
  .data (size: 32)




🧩 7. Linker (rrl)


The rrl linker combines multiple .o files into a static binary or ELF/PE executable.


Features




Deterministic section ordering


No timestamps, UUIDs, or non-reproducible metadata


Optional checksum validation




Example


rrl main.o libio.o -o firmware.elf



Output Options




Option
Output Type
Description




--format=elf64
ELF
Linux target


--format=pe64
PE
Windows target


--format=bin
Raw binary
Bare-metal


--deterministic

Forces hash-identical builds





⚙️ 8. Backend Components




Component
Responsibility




rir_reader
Parses RIR text format


rir_translator
Converts RIR ops to instruction templates


asm_emitter
Writes NASM code


rra
Assembles NASM to object


rrl
Links object to final binary





🧩 9. Determinism System


RR backend uses Content-Deterministic Emission (CDE):




Instruction order derived solely from AST order


No random seeds or compiler timestamps


Identical source always → identical binary hash




Verification


rrc main.rr --verify-hash
Binary SHA256: 17E8C3F1A9...
Reproduced build: identical ✅




🧠 10. Backend Architecture Diagram


         ┌────────────┐
         │   RIR IR   │
         └─────┬──────┘
               │
               ▼
     ┌──────────────────┐
     │ Instruction Map  │
     └─────┬────────────┘
           │
           ▼
   ┌─────────────────┐
   │  ASM Generator  │─────▶ .asm
   └─────────────────┘
           │
           ▼
   ┌─────────────────┐
   │   Assembler     │─────▶ .o
   └─────────────────┘
           │
           ▼
   ┌─────────────────┐
   │     Linker      │─────▶ .bin / .elf / .exe
   └─────────────────┘




🧭 11. Backend Roadmap




Phase
Feature
Status




Phase 1
NASM text emitter
🟢 Complete


Phase 2
Minimal assembler (rra)
🔵 In progress


Phase 3
ELF/PE linker (rrl)
🔵 Planned


Phase 4
Bare-metal binary emitter
⚫ Future


Phase 5
Backend optimization freeze
🚀 Release-ready





🏁 12. Summary


The RR backend defines a new compiler paradigm:

reproducibility and transparency come before optimization.






No LLVM


No hidden passes


No shifting semantics






Every instruction RR emits is intentional, permanent, and explainable.

This is not a compiler that guesses — it’s one that remembers.


RR Backend = Predictability Engine.



---


