# 📘 RR Language Specification (Draft v0.1)

## 🧭 Overview
**RR** is a statically typed, deterministic compiled language designed for **industrial reliability**, **embedded systems**, and **long-term reproducibility**.  
It aims to coexist with Rust while replacing it in industrial environments where **predictability**, **stability**, and **compiler independence** are critical.

RR syntax draws conceptual familiarity from Rust and C-family languages, but removes dynamic borrowing and complex lifetime tracking — replacing them with **region-based deterministic ownership**.

---

## 🧩 Language Fundamentals

### File Structure
Each `.rr` file represents a **module unit**.  
Compilation starts from an entry function `fn main()`.

```rr
fn main() {
    let msg: text = "Hello, Industry!";
    print(msg);
}




🧱 Syntax Overview


Comments


// Single-line comment
/* Multi-line comment */



Functions


fn add(a: int, b: int) -> int {
    return a + b;
}





Functions are pure by default.


Side effects must be explicitly annotated with unsafe or io.





🔡 Variables


let x: int = 10;
let msg: text = "Hello!";
mut counter: int = 0;





let defines immutable bindings.


mut allows modification, but restricted to safe regions.


Explicit type declarations are mandatory unless inferred from literals.





🧭 Control Flow


if x > 10 {
    print("High");
} else {
    print("Low");
}

while n < 10 {
    n += 1;
}



No implicit conversions between numeric or boolean types.



🧩 Regions (Lifetime Control)


region main {
    let buffer: [byte; 256];
    init(buffer);
}





A region defines a compile-time memory scope.


No runtime GC, no dynamic borrow checking.


All variables are freed deterministically when the region exits.





⚙️ Type System




Category
Example
Description




Primitive
int, float, bool, text, byte
Base data types


Composite
struct, enum, union
User-defined


Pointer
ptr<T>
Raw pointer, immutable by default


Array
[T; N]
Fixed-size array


Region
region
Compile-time memory scope


Function
fn(T) -> U
Typed function definition





🧰 Structs and Enums


struct Point {
    x: int,
    y: int,
}

enum Color {
    Red,
    Green,
    Blue,
}



Structs are immutable by default.

Mutable regions must be declared explicitly.



🔒 Safety Model — Region Ownership


RR replaces the borrow checker with region ownership:




Ownership is scoped by region blocks.


No dangling references — compile-time verified.


No heap allocation outside designated regions.




Example:


region task {
    let port: ptr<Device>;
    port.write(0xFF);
}



If a pointer escapes its region → compilation error.



⚡ Concurrency


RR supports compile-time safe concurrency via message-passing threads.


spawn worker(task: fn()) -> thread;
channel<int> tx, rx;
tx.send(42);
rx.recv();



All thread communication is type-checked and region-bound.



🧭 Modules and Imports


module math {
    pub fn add(a: int, b: int) -> int { return a + b; }
}

use math::add;

fn main() {
    let result = add(2, 3);
}





Modules are file-level units.


Public items require pub.


No macro system by default — everything is explicit.





🧩 Error Handling


Errors are handled deterministically using tagged unions.


fn divide(a: int, b: int) -> Result<int, text> {
    if b == 0 { return Err("Division by zero"); }
    return Ok(a / b);
}



try expressions are supported:


let result = try divide(10, 2);




🧠 Compile-Time Execution


RR allows constexpr evaluation for static computations:


constexpr fn square(x: int) -> int { return x * x; }
const SIZE: int = square(4);




🧩 Future Language Features




consteval embedded-safe constant folding


Hardware-mapped I/O support (port, mmio primitives)


NASM backend optimization hints


Formal verification toolchain integration





📦 Standard Library (Planned)




Module
Purpose




core
Basic types, intrinsics


io
Input/output abstraction


math
Deterministic math routines


thread
Safe concurrency primitives


sys
System and hardware access





🏁 Summary


RR is designed for environments where:




Long-term reproducibility > constant evolution


Deterministic behavior > dynamic abstraction


Safety and performance must both be guaranteed






“Rust protects you from mistakes.

RR prevents them by design.”





---
