# 🧠 RR Compiler Architecture
> Design Specification (Draft v0.1)

---

## 🎯 Goal
RR aims to build a **fully self-contained, LLVM-independent compiler** capable of generating reproducible machine code for industrial systems.  
The RR compiler prioritizes **determinism**, **transparency**, and **long-term stability** over optimization complexity.

---

## 🧩 Pipeline Overview




RR Source (.rr)
↓
Lexer → Parser → AST → Semantic Analyzer
↓
RIR (RR Intermediate Representation)
↓
Code Generator (NASM / ELF / PE)
↓
Assembler + Linker
↓
Executable / Object File



---

## 🧱 Core Components

### 1️⃣ Lexer
- Converts source text into token streams.
- Token types: identifiers, keywords, literals, symbols.
- Contextual keyword resolution (e.g., `mut`, `region`, `constexpr`).
- No preprocessor, macros, or conditional compilation.

Example token output:



fn [IDENT] add [SYMBOL] ( a : int , b : int ) { return a + b ; }



---

### 2️⃣ Parser
- Predictive LL(2) parser with deterministic grammar.
- Error-tolerant recovery for industrial debugging.
- Constructs a **flat Abstract Syntax Tree (AST)** for fast traversal.

Grammar sample (simplified EBNF):



Program ::= { FunctionDecl }
FunctionDecl ::= "fn" IDENT "(" [Params] ")" ["->" Type] Block
Block ::= "{" { Statement } "}"
Statement ::= LetDecl | Expr | IfStmt | WhileStmt | ReturnStmt



---

### 3️⃣ AST (Abstract Syntax Tree)
AST is structured for **static region ownership analysis** and **semantic clarity**.

Example:
```rr
fn main() {
    region io {
        let msg: text = "Hello";
        print(msg);
    }
}



AST node (simplified JSON form):


{
  "type": "Function",
  "name": "main",
  "body": [
    {
      "type": "Region",
      "name": "io",
      "statements": [
        { "type": "Let", "name": "msg", "value": "Hello" },
        { "type": "Call", "name": "print", "args": ["msg"] }
      ]
    }
  ]
}




4️⃣ Semantic Analyzer


Responsible for:




Type inference and validation


Region lifetime verification


Function purity checks


Constexpr propagation




All analysis is compile-time only, no runtime reflection or borrow tracking.



5️⃣ RIR (RR Intermediate Representation)


RIR is a three-address, SSA-like IR optimized for deterministic code generation.


Example:


%1 = const "Hello"
%2 = call print %1
ret 0



RIR is human-readable and fully reversible → IR can regenerate .rr source.


RIR layers:




Layer
Purpose




RIR-Core
Arithmetic & logic


RIR-Mem
Region memory ops


RIR-Thread
Concurrency & messaging


RIR-IO
Hardware I/O operations





6️⃣ Code Generator


Goals:




Simple, reproducible machine code generation


NASM text output by default


Optionally emits ELF/PE directly (no external assembler)




Supported targets:




Backend
Format
Status




NASM
.asm
🟢 Active


ELF (Linux)
.o
🔵 Planned


PE (Windows)
.obj
🔵 Planned


BIN (bare-metal)
.bin
⚫ Future





7️⃣ Assembler & Linker


RR includes a minimal built-in assembler (rra) and static linker (rrl).


Features:




Deterministic instruction layout


Zero hidden sections


Reproducible binary output (identical hash guaranteed)





🧰 Toolchain Components




Tool
Role




rrc
Main compiler frontend


rrfmt
Code formatter


rrdoc
Documentation generator


rrpkg
Lightweight package manager (offline)


rra
Assembler


rrl
Static linker





🧬 Language Runtime


RR runtime (rrrt) is optional — used only for I/O and threading.




No garbage collector


No dynamic allocation (unless declared in region)


All memory is stack- or region-managed





⚙️ Build Example


rrc main.rr -o main.asm
rra main.asm -o main.o
rrl main.o -o main.bin



Or in one step:


rrc main.rr --target=bare --output=main.bin




🧭 Compiler Stages Roadmap




Phase
Description
Status




Phase 1
Lexer + Parser + AST
🟢 Active


Phase 2
Semantic analyzer + RIR
🔵 Planned


Phase 3
NASM backend + assembler
⏳ In progress


Phase 4
ELF/PE support
⚫ Future


Phase 5
Self-hosting RR compiler
🚀 Target





🏁 Summary


RR compiler is designed to build itself, remain independent, and stay unchanged for decades —

ensuring every binary built today can be reproduced bit-for-bit in 2045.




“Where compilers evolve, RR endures.”





---
