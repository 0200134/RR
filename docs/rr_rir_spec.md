⚙️ rr_rir_spec.md


RR Intermediate Representation (RIR) Specification — Readable, Reversible, Deterministic


# RR Intermediate Representation (RIR) Specification
> Readable, Reversible, and Deterministic  
> Version 1.0 — R3C Foundation / RR Compiler Division

---

## 🧭 Overview
The **RR Intermediate Representation (RIR)** is a low-level, deterministic representation of RR source code.  
It bridges the compiler’s *semantic analysis* and *assembly generation* phases.

Unlike LLVM IR, RIR is designed to be:
- **Human-readable**
- **1:1 reversible** (back to `.rr`)
- **Fully deterministic** — same source always produces the same RIR
- **Independent of LLVM or platform backends**

---

## ⚙️ Compiler Flow




RR Source (.rr)
↓
Semantic Parser
↓
RIR Generator  ← (this document defines this layer)
↓
NASM Emitter
↓
Assembler (rra)
↓
Binary (ELF / BIN)



---

## 🧩 1. Structure of an RIR File

### Basic Layout



FUNC 

END



Example:



FUNC main
%1 = CONST "Hello"
%2 = CALL print %1
RET 0
END



---

## 🧱 2. Identifiers and Registers

| Symbol | Meaning |
|---------|----------|
| `%N` | Virtual register (temporary value) |
| `@L` | Label (jump target) |
| `FUNC` | Function start |
| `END` | Function end |

Each `%N` is immutable once assigned — RIR is purely SSA (Static Single Assignment).

---

## 🔧 3. Instruction Set

| Opcode | Operands | Description |
|---------|-----------|--------------|
| `CONST` | value | Define literal or constant |
| `MOV` | dst, src | Copy value |
| `ADD` | a, b | Integer addition |
| `SUB` | a, b | Integer subtraction |
| `MUL` | a, b | Multiplication |
| `DIV` | a, b | Division (safe-checked) |
| `CMP` | a, b | Compare |
| `JMP` | label | Unconditional jump |
| `JZ` / `JNZ` | label | Conditional jump |
| `CALL` | func, args | Function call |
| `RET` | value | Return value |
| `LOAD` | dst, addr | Read memory |
| `STORE` | addr, src | Write memory |
| `NOP` | — | No operation |

---

## 🧠 4. Example — Simple Function

Source:
```rr
fn add(a: int, b: int) -> int {
    return a + b;
}



RIR:


FUNC add
    %1 = ARG a
    %2 = ARG b
    %3 = ADD %1, %2
    RET %3
END




🧩 5. Example — Branching


Source:


fn check(x: int) {
    if x > 0 { print("pos"); }
    else { print("neg"); }
}



RIR:


FUNC check
    %1 = ARG x
    %2 = CONST 0
    %3 = CMP %1, %2
    JZ @NEG
    CALL print "pos"
    JMP @END
@NEG:
    CALL print "neg"
@END:
    RET 0
END




🧱 6. Example — Region Awareness


Each region is explicitly tracked in RIR metadata.


FUNC main
#region: io
    %1 = ALLOC [byte; 256]
    CALL driver_write %1
    DEALLOC %1
END



RIR guarantees that every allocation has a matching deterministic DEALLOC before END.



🔒 7. Determinism Rules




Instruction ordering mirrors AST traversal.


Identical source = identical instruction sequence.


Compiler inserts no hidden optimizations.


All constants and labels are sorted deterministically.




Build verification command:


rrc main.rr --verify-rir



Output:


RIR verification passed ✅
Hash: 9A6E4C... identical




⚙️ 8. Function Metadata Section


Each RIR function includes a deterministic metadata header:


#meta: name=main
#meta: params=0
#meta: region=global
#meta: checksum=0xA13B5



Used for reproducibility and backend linkage (rra → rrl).



🧩 9. Comments and Annotations


RIR allows inline comments for readability:


%1 = CONST 42      # literal constant
%2 = CALL log %1   # call log function



Compiler ignores comments during verification.



🧠 10. Binary RIR Encoding (Optional)


Future versions may emit .rirb binary form.




Section
Content




Header
magic bytes + version


IR Data
opcodes + operands


Metadata
region + hash


Footer
checksum




Hash-verified and byte-identical across builds.



🧩 11. Verification and Hashing


Each .rir file produces a SHA256 hash for reproducibility:


rir_hash(main) = SHA256(instr_sequence)



Compiler verification:


rrc --verify main.rr
✔ RIR identical
✔ ASM identical
✔ Binary identical




🧭 12. Comparison Table




Feature
LLVM IR
RIR




Human-readable
❌ No
✅ Yes


Deterministic
⚠️ Partial
✅ Complete


SSA Form
✅
✅


Debug-friendly
❌
✅


Backend independence
❌
✅


Reversible to source
❌
✅





✅ Summary


RIR is the deterministic heart of the RR compiler:

human-readable, reversible, and fully reproducible across machines.




“LLVM optimizes.

RIR remembers.”




RR Intermediate Representation = Reproducibility in Code Form.



📜 License


MIT License © 0200134 — R3C Foundation



---
