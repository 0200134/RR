# ⚙️ RR Runtime Design Document  
> RR Runtime Architecture (RRRT) — Deterministic Industrial Runtime  
> Version 0.1 (Internal Draft)

---

## 🧭 1. Overview

RR runtime (RRRT) is a **minimal, deterministic execution layer** designed to support:
- Region-based memory control  
- Predictable I/O  
- Compile-time verified concurrency  
- Platform-agnostic operation  

Unlike modern managed runtimes, RRRT provides **no dynamic garbage collection**, **no reflection**, and **no JIT**.  
Every allocation, thread, and system interaction is **resolved deterministically** at compile time or region initialization.

---

## 🧩 2. Design Principles

| Principle | Description |
|------------|-------------|
| **Determinism** | Every operation produces the same result for identical input. |
| **Minimalism** | No unnecessary layers — the runtime can fit in <100KB. |
| **Transparency** | Every runtime call maps to explicit system or hardware instructions. |
| **Isolation** | Each thread or I/O task is scoped to a `region` with strict ownership. |
| **Reproducibility** | RRRT versions are frozen; behavior never changes after release. |

---

## 🧱 3. Runtime Layers




+---------------------------------------------------+
| Application (RR Code)                             |
|  fn main() { ... }                                |
+---------------------------------------------------+
| RRRT Core (Memory, Thread, IO, Region Manager)    |
+---------------------------------------------------+
| Platform Layer (Syscalls / HAL / Bare-metal ops)  |
+---------------------------------------------------+
| Hardware                                          |
+---------------------------------------------------+



---

## 🧠 4. Memory Model

### 4.1 Region-Based Allocation
Memory is divided into **regions**, each managed deterministically by the compiler.

```rr
region io {
    let buffer: [byte; 512];
    driver_write(buffer);
}





All variables in a region are stack-allocated or statically reserved.


No dynamic heap by default.


The region is destroyed automatically at block exit.




4.2 Allocator


RRRT implements a simple static-region allocator:




Compile-time layout (no runtime allocation)


Optional heap region for controlled dynamic memory


Configurable memory pools for embedded targets




Allocator structure:


RegionHeader {
    id: u16,
    size: u32,
    used: u32,
    base: *byte
}




⚙️ 5. Concurrency Model


5.1 Overview


RR provides compile-time verified threads, controlled by RRRT at runtime.

No shared mutable state exists outside declared channels.


5.2 Example


fn worker(id: int) {
    print("Worker:", id);
}

fn main() {
    let t1 = spawn worker(1);
    let t2 = spawn worker(2);
    join(t1);
    join(t2);
}



5.3 Thread Lifecycle




Phase
Description




Spawn
Thread is created via static table allocation


Run
Executed on cooperative scheduler


Join
Parent waits deterministically


Terminate
Memory region automatically reclaimed




Thread control block (TCB):


Thread {
    id: u32,
    region: *RegionHeader,
    state: ENUM { Ready, Running, Done },
    stack_base: *byte,
}




🔌 6. I/O Model


RRRT provides a deterministic I/O subsystem — no buffering, no OS-dependent behavior.


API Example


io::write(port: int, data: [byte]);
io::read(port: int, buffer: [byte]);





Calls map directly to hardware registers or OS syscalls.


On embedded targets, RRRT bypasses libc and performs direct MMIO (memory-mapped I/O).




Example mapping:




Function
Native Call
Description




io::write
OUT dx, al (x86)
Port-mapped I/O


io::read
IN al, dx
Read from device





💾 7. File System (Optional Layer)


For hosted systems, RRRT provides a simple POSIX-like FS abstraction.


use fs::*;

fn main() {
    let file = fs::open("log.txt");
    file.write("Hello, RRRT!");
}



Implementation goal:




Deterministic ordering (no background buffering)


Checksummed file metadata


Region-tied file handles (auto-closed on region exit)





🧩 8. Scheduler Design


RRRT uses a cooperative deterministic scheduler.

Each thread yields voluntarily; no preemption.


Scheduler Loop


while (true) {
    for (t in threads) {
        if (t.state == READY) run(t);
    }
}





No OS dependency (can run on bare metal).


Behavior identical across hardware targets.


Each thread executes within a memory region.





🧠 9. Exception & Panic Model


RR does not implement exception unwinding.

Instead, RRRT uses error codes + static fallback regions.


fn divide(a: int, b: int) -> Result<int, text> {
    if b == 0 { return Err("Divide by zero"); }
    return Ok(a / b);
}



Runtime panic = fatal deterministic abort:


RRRT Panic: divide by zero
  at main.rr:42
  region: core




🧩 10. Runtime Configuration


RRRT can be built in three profiles:




Profile
Description




Bare
For embedded / microcontrollers, no heap or FS


Static
For OS-level binaries with file I/O


Full
Includes thread and I/O system




Example:


rrc main.rr --runtime=bare
rrc main.rr --runtime=static
rrc main.rr --runtime=full




🧰 11. Runtime Modules




Module
Responsibility




rr_mem
Region allocator, static pools


rr_thread
Cooperative thread scheduler


rr_io
Hardware and OS I/O


rr_fs
Minimal filesystem layer


rr_sys
Platform abstraction (syscalls, HAL)





🧩 12. Platform Abstraction Layer (PAL)


The PAL isolates platform dependencies under a unified interface.




Function
Linux
Windows
Bare-Metal




sys_write
write(fd, buf, len)
WriteFile()
OUT port, reg


sys_read
read(fd, buf, len)
ReadFile()
IN reg, port


sys_thread
pthread_create()
_beginthreadex()
Static table




This makes RRRT portable while maintaining strict determinism.



🧭 13. Verification & Determinism Testing


RRRT includes a determinism verifier:


rrrtd --verify main.bin
✔ hash match (binaries identical)
✔ region lifetime validated
✔ thread table consistent



All builds must produce bit-identical binaries across machines.



🏁 14. Summary


RRRT is not a runtime in the modern sense —

it is a deterministic execution core, ensuring that:




No memory leaks can exist by design.


No thread executes outside its region.


No runtime behavior changes between builds.






“RRRT doesn’t run your program — it preserves it.”




RRRT — The Deterministic Heart of RR.



---
