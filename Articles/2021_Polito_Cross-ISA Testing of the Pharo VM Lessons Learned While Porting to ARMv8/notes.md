<!-- Please prefix the notes with the date as in [22/12/2020] -->

[30/06/2021]

**Pharo VM:** At its core, the Pharo VM contains a threaded bytecode interpreter, a linear non-optimising JIT compiler named **Cogit** and a generational scavenger garbage collector (copy collector for young objects and mark-compact collector for older objects). The VM itself contains **255 bytecodes (in 77 families)**, **340 primitive methods** with some duplicated in the interpreter and the JIT compiler as well as **150 IR instructions**.

**Slang:** Smalltalk-to-C VM-specific transpiler. It transpiles a group of classes into a single C file. Methods are translated into functions, message sends to function calls and polymorphism is forbidden. It helps introducing interpreter optimizations (localisation of critical variables, inlining of bytecode cases, threaded code) and helps simulating the Pharo VM by executing it as normal code.

**Cogit:** The JIT compiler is a **method-based non-optimising linear** JIT compiler. It uses a linear **2-address-code** immediate representation. Compiling a method includes three phases: (1) *bytecode scan phase* extracts meta-data (i.e. need for frame) (2) *bytecode parse phase* does an abstract interpretation through a stack-bytecode to register-IR transformation (3) *code generation phase* computes the IR instruction offsets and assembles the machine code for the platform.

**CogRTL IR:** The intermediate representation uses a fixed number of virtual registers (such as `ReceiverRegister` or `ClassRegister`). These registers are needed by the compilation and are mapped to the real registers ahead of time as a compiler configuration for each supported backend/platform. CogRTL aims to be as machine-independent as possible, there is a clear separation between the compiler's frontend that parses bytecode and generates an IR and the compiler's backend that generates machine code from the IR.

**Code Patching and PICs:** Apart from method compilation, Cogit performs code patching as implemented in the compiler's backend. This does not use any of the support from CogRTL and is used in two scenarios: mono/poly/mega-morphic inline caches and updating object references when moved by the garbage collector.