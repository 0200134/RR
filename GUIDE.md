# 🧩 RR — Rust Replacer Language Guide

## Introduction

**RR (Rust Replacer)** is a next-generation **industrial-grade systems programming language** under the **R3C Foundation**.  
It is designed to **coexist with Rust** while **replacing Rust in environments where Rust cannot sustain long-term reproducibility, stability, and deterministic compilation**.

RR focuses on:
- **Compiler sovereignty** — no external LLVM dependency  
- **Metal-level determinism** — predictable low-level behavior  
- **Long-term sustainability (LTSS)** — decades-long reproducible toolchains  

---

## 1. Installation & Build

RR is currently in early development under the R3C Foundation.  
Build scripts will be released gradually with the compiler prototype.

### 🔧 Prerequisites
- CMake ≥ 3.25  
- Rust ≥ 1.80 (for bootstrap only)  
- Python ≥ 3.10  
- NASM or YASM (for backend assembly)  
- Git & GNU Make  

### 🏗️ Build (planned workflow)
```bash
git clone https://github.com/r3c-foundation/RR.git
cd RR
cmake -B build
cmake --build build
./build/bin/rrc --version





The initial compiler rrc will be a self-hosted tool written in Rust,

then progressively rewritten in RR itself (bootstrapping phase).





2. Language Overview


RR aims to provide C-level determinism with Rust-level safety,

bridging the gap between legacy embedded C systems and modern secure toolchains.




Goal
Description




Replace Rust in industrial contexts
Focused on reproducibility and predictable compilation rather than iteration speed.


Long-term toolchain stability
Designed to compile identically even decades later.


LLVM-independent pipeline
Built under the R3C LLVM-Zero Ecosystem.


Full NASM/ELF/PE backend
Enables direct and transparent metal-level control.





3. Core Philosophy


RR was born from a simple belief:




“Safety and determinism should not depend on a third-party compiler.”




🧱 Principles




Reproducibility > Performance


Transparency > Abstraction


Sovereignty > Dependence




🧭 Coexistence with Rust


RR is not an anti-Rust project.

It coexists with Rust and complements it for industrial or embedded contexts

where toolchain longevity and reproducible binaries are mission-critical.



4. Repository Structure


RR/
├── docs/                  # Language design & architecture documents
├── rr_spec.md             # Language specification
├── rr_design.md           # Design philosophy & structure
├── rr_backend_design.md   # NASM/ELF/PE backend design
├── rr_compiler_design.md  # Compiler pipeline & architecture
├── rr_runtime.md          # Runtime and deterministic execution layer
├── README.md              # Overview
└── LICENSE                # MIT License




5. Contributing


RR welcomes contributions aligned with the R3C Foundation’s long-term vision.


🧠 Contribution Principles




Sustainability — Code must compile deterministically.


Transparency — Avoid hidden dependencies.


Self-hostability — Every component should be self-replacing over time.


Simplicity — Minimalist syntax and modular architecture.




💡 Getting Started




Fork the repository


Open a discussion or issue under GitHub Discussions


Submit a pull request with clear intent and reproducible results





6. Roadmap (Draft)




Phase
Year
Description




Phase 0
2025
Design, specs, backend model (NASM/ELF)


Phase 1
2026
Prototype compiler (Rust-based bootstrap)


Phase 2
2027
Self-hosted RR compiler


Phase 3
2028+
LTSS stabilization and full ecosystem integration





7. License


RR is distributed under the MIT License.

See LICENSE for details.



8. Contact


R3C Foundation

Official repository: https://github.com/r3c-foundation




“Rewrite the base.

Build compilers that heal themselves.”





9. Toolchain Installation & Usage


RR provides a fully independent compiler toolchain designed for long-term reproducibility and deterministic builds.

Unlike traditional Rust or LLVM-based systems, RR’s toolchain does not rely on LLVM or Cargo, but instead uses a self-contained compiler pipeline.



🧰 Toolchain Components




Tool
Description




rrc
The RR Compiler — compiles .rr source code into NASM, ELF, or PE targets.


rrrun
The RR Runtime Executor — runs deterministic binaries within controlled environments.


rrstd
The RR Standard Library — minimal, predictable runtime components with no undefined behavior.


rrpkg (planned)
Package & dependency manager designed for long-term reproducible builds.


rrdoc (planned)
Documentation generator for the RR ecosystem.





⚙️ Installation (Planned Workflow)


Once released, the toolchain can be installed via standard build or package scripts:


From Source


git clone https://github.com/r3c-foundation/RR.git
cd RR
cmake -B build
cmake --build build
sudo cmake --install build



Verify Installation


rrc --version
rrrun --help




🏗️ Compiler Pipeline Overview


The RR toolchain executes a fully reproducible pipeline as follows:


[Source Code] → [RR-IR] → [Backend: NASM / ELF / PE] → [Deterministic Binary]



Each stage is deterministic, meaning:




No randomization or time-dependent variables in builds


Identical outputs across machines, OSes, or years later


Bitwise identical binaries for reproducibility validation





🧩 Example Workflow


1. Create a new RR project


mkdir hello_rr
cd hello_rr
echo 'fn main() { print("Hello RR!"); }' > main.rr



2. Compile


rrc main.rr -o hello



3. Run


rrrun hello



Output:


Hello RR!




🔄 Target Formats


RR supports multiple backend targets through the R3C LLVM-Zero backend:




Target
Description




NASM
Raw x86_64 Assembly output for full transparency.


ELF
Linux-compatible executable format (deterministic sections).


PE
Windows-compatible executable (Portable Executable).


OBJ (planned)
Object file for static linking into C/C++ pipelines.






RR intentionally avoids dynamic linking and non-deterministic optimizations.

Its priority is determinism, not runtime variance.





🧱 Integration with R3C Ecosystem


RR is designed to integrate seamlessly with the broader R3C compiler pipeline:


C/C++ → Rust → RR → NASM → ELF/PE





Upstream Compatibility — Rust → RR transpilation supported


Downstream Stability — NASM deterministic backend output


Cross-platform Reproducibility — identical binaries across OS targets





🔒 Long-Term Support (LTSS Toolchain)


RR releases are versioned as part of the R3C-LTSS model:




Every major release is frozen for 10 years minimum


Reproducible builds guaranteed via deterministic toolchain snapshot


rr-ltss tool (planned) will verify build hashes and IR integrity





🧩 Example LTSS Command (Future)


rr-ltss verify --project hello_rr --hash a3f7c19e
rr-ltss restore --version 1.2-LTSS




🧠 Summary


RR’s toolchain represents a new era of compiler design:




LLVM-independent


Deterministic


Self-hosting


Future-proof for decades






“If a compiler can reproduce itself perfectly,

it no longer depends on anyone — it becomes sovereign.”





---



