# ⚙️ RR Compiler Design Document  
> Version 0.1 — Architectural Blueprint  

---

## 🧭 1. Design Philosophy
RR compiler (codename `rrc`) is built around **determinism**, **readability**, and **self-containment**.  
Unlike LLVM-based compilers, `rrc` does not depend on complex optimization layers or external code generation APIs.  
It aims to produce **reproducible binaries** and a **transparent build pipeline**, suitable for **industrial LTSS** (Long-Term Support Systems).

---

## 🧩 2. High-Level Pipeline




Source (.rr)
↓
[1] Lexer
↓
[2] Parser
↓
[3] AST Builder
↓
[4] Semantic & Region Analyzer
↓
[5] RIR Generator (Intermediate Representation)
↓
[6] Optimizer (deterministic, optional)
↓
[7] Backend Code Generator (NASM / ELF / PE)
↓
[8] Assembler + Linker
↓
Executable / Object / Static Binary



Each stage must be **pure**, **deterministic**, and **debuggable** without external tools.

---

## 🧱 3. Core Compiler Modules

| Module | Purpose | Output |
|--------|----------|--------|
| **Lexer** | Tokenizes `.rr` source | Token stream |
| **Parser** | Builds grammar tree | Abstract Syntax Tree (AST) |
| **AST** | Represents language structure | In-memory nodes |
| **Semantic Analyzer** | Type, region, scope, and lifetime validation | Annotated AST |
| **RIR Generator** | Converts AST → RR Intermediate Representation | `.rir` IR |
| **Backend** | Translates RIR → assembly or binary | `.asm`, `.o`, `.bin` |
| **Assembler & Linker** | Merges objects and fixes symbols | Executable |

---

## 🔤 4. Lexer Design

### Key Requirements
- Deterministic tokenization (no regex ambiguity)
- Contextual keyword resolution
- Support UTF-8 identifiers
- No macro expansion

### Example Implementation (pseudo-code)
```cpp
Token next_token() {
    skip_whitespace();
    if (match_alpha()) return identifier_or_keyword();
    if (match_digit()) return number_literal();
    if (match_symbol()) return symbol_token();
    error("unexpected character");
}



Token Types


IDENT, INT_LIT, FLOAT_LIT, STR_LIT,
FN, LET, MUT, REGION, RETURN,
IF, ELSE, WHILE, STRUCT, ENUM, MODULE, USE,
LBRACE, RBRACE, LPAREN, RPAREN, COLON, SEMI, ARROW, EQ, PLUS, MINUS, STAR, SLASH




🧩 5. Parser Design


Parsing Style




Recursive descent with manual stack


No parser generators


Error recovery via predictive lookahead (LL(2))


Grammar: frozen once stable (no breaking changes)




Example Grammar (Simplified)


Program ::= { FunctionDecl | StructDecl | EnumDecl }
FunctionDecl ::= "fn" IDENT "(" [Params] ")" ["->" Type] Block
Block ::= "{" { Statement } "}"
Statement ::= LetDecl | ExprStmt | ReturnStmt | RegionDecl



AST Node Example (C-like representation)


struct ASTFunction {
    string name;
    vector<ASTParam> params;
    ASTType return_type;
    ASTBlock body;
};




🧠 6. Semantic Analysis


Purpose




Resolve symbols and scopes


Verify region lifetime constraints


Validate type safety


Mark unsafe regions


Propagate constants for constexpr




Algorithm Outline




Build symbol table


Attach type and region annotations to AST nodes


Check all variables are defined before use


Verify no pointer escapes its region


Resolve imports and module scopes




Example Rule


[RegionEscapeCheck]
If variable of type ptr<T> is declared inside region R,
it cannot be returned or assigned to a variable outside R.




🔧 7. RIR (RR Intermediate Representation)


RIR (RR Intermediate Representation) is designed to be stable and reversible, meaning it can regenerate .rr source perfectly.


Structure


FUNC main
    %1 = CONST "Hello"
    %2 = CALL print %1
    RET 0
END



Features




SSA-like form


Deterministic instruction ordering


Human-readable, text-based


Target-independent




Example Mapping




RR Source
RIR Output




let x: int = 10;
%1 = CONST 10


x + 5
%2 = ADD %1, 5


print(x)
%3 = CALL print, %1





⚙️ 8. Backend Code Generation


Backend Goals




Minimal, deterministic NASM code output


Each RIR instruction maps 1:1 to ASM block


No peephole optimization that affects binary reproducibility




Example NASM Output


section .text
global main
main:
    mov rdi, msg
    call print
    mov rax, 0
    ret
section .data
msg db "Hello, Industry!", 0



Supported Targets




Target
Output
Status




x86_64-nasm
.asm
✅ Active


elf64
.o
🔵 Planned


pe64
.obj
🔵 Planned


baremetal
.bin
⚫ Future





🧩 9. Optimizer (Optional)


The optimizer performs non-destructive optimizations only:




Constant folding


Dead code elimination (deterministic)


Inline constexpr evaluation


Region lifetime compression




Optimizations are verifiable — the compiler can regenerate the pre-optimized IR for debugging.



🧰 10. Toolchain Components




Tool
Function




rrc
Main compiler


rra
NASM-compatible assembler


rrl
Linker


rrfmt
Code formatter


rrdoc
Documentation generator


rrpkg
Package manager (offline mode only)





🧩 11. Error Model


RR compiler errors are deterministic and traceable.


Example:


E201: pointer escape from region [io] → [global]
  --> main.rr:12:9
   |
12 |     return port;
   |            ^ cannot return pointer from local region



Each diagnostic message includes:




Error code


Region context


Suggested fix





🧱 12. Testing & Self-Hosting Roadmap




Stage
Objective
Deliverable




Phase 1
Implement lexer, parser, AST
rrc --dump-ast


Phase 2
Add semantic + RIR generator
rrc --emit-rir


Phase 3
Add NASM backend + assembler
rrc main.rr -o main.asm


Phase 4
Add ELF/PE backend
rrc main.rr --target=elf64


Phase 5
Self-host RR compiler
rrc (written in RR)





🧬 13. Determinism Contract


Every binary built from RR must satisfy:




Bit-identical output for same source, same compiler version.


Compiler version frozen after release.


No hidden compiler state or timestamps.


Fully offline reproducibility.






A compiler that changes your binary without your consent is a liability.





🏁 14. Summary


RR Compiler (rrc) is designed to:




Be self-hosting, stable, and transparent


Produce verifiable, reproducible binaries


Serve as a foundation for industrial LTSS software stacks






“Where LLVM depends, RR defines.

Where Rust evolves, RR endures.”





---
