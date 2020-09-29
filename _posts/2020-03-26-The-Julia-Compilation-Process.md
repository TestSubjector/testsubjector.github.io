---
layout: post
title: "The Julia Compilation Process"
date: 2020-03-26
mathjax: true
---
----------------
A look at how Julia works in the background.

----------------
##### JIT Compilation
----------------

Julia has a just-in-time (JIT) compilation. This means that the code is
dynamically compiled during the execution of the program, also known as
the program run time. In this way the previous step of compiling the
code into an executable is completely excluded from consideration.

The idea behind JIT compilation is to bring the benefits of both
(static) compilation and interpretation.

JIT compilers can also do dynamic recompilation which is targeted
recompilation of certain sections of the codebase when the compiler
believes that the newly generated code will be more efficient, based on
information not available to a traditional static compiler. This
information in most cases is the run time environment in which the
program executes.

How does the run time environment matter?

During run time, interpreters have access to input parameters, control
flow, and target machine specifics. This information may change from run
to run or be unobtainable prior to run-time. Additionally, gathering
some types of information about a program before it runs may involve
algorithms which are undecidable using static analysis. [^1]

----------------
##### Warmup delay
----------------

Due to the time taken by the JIT compiler to initially load the code and
compile it for the first time during an execution, there is an initial
lag in the runtime of a program. The Julia programming language, as a
dependent of the JIT compiler also accrue these delays. This is the main
reason for the repeated advice to do a *warmup call* on any segment of
the code before it is benchmarked, so as to avoid incurring the
compilation time in the benchmark statistics.

It is important to note that there are major differences between the
working of Julia's JIT compiler and that of say Python. The compilation
target of Python is specialised bytecode which is then interpreted by
the CPython interpreter.

In the case of Julia, there are four major level of disassembly steps
which transforms the source code directly to native machine code. This
adds to the compilation complexity, but in return helps Julia gain it's
much vaunted speed.

----------------
##### Julia's Four Level Disassembly Compilation
----------------
We introduce the various stages of Julia compilation, their purpose and
a small insight into how to utilise the information given by them. Each
stage of Julia compilation has its own lowered form, an intermediate
representation of the original code in question.

Let us declare and then call a test method *pos(x)* which will return
the input argument x if x is greater than zero, otherwise it will return
zero in the same type (integer/float etc) as that of input x.

```julia
    julia> pos(x) = x < zero(x) ? zero(x) : x
    pos (generic function with 1 method)
```
<br><br/>
The first step is the construction of the lowered code form, which is
used by the further type inference and code generation processes. In the
lowered form there are fewer types of nodes, all macros are expanded,
and all control flow is converted to explicit branches and sequences of
statements. This means transforming the method from Julia's high-level
syntax (for example iterative statements or ternary operators) to a
smaller set of common primitives (for example gotos).

```julia
    julia> @code_lowered pos(1)
    CodeInfo(
        1 _ %1 = Main.zero(x)
        _   %2 = x < %1
        ___      goto #3 if not %2
        2 _ %4 = Main.zero(x)
        ___      return %4
        3 _      return x
    )
```
<br><br/>
The typed inferred incarnation of the code is similar to the lowered
form, but with expressions annotated with type information and some
generic function calls replaced with their implementations. The
code\_typed macro presents a method implementation for a particular set
of argument types after type inference and inlining. More on this later.

```julia
    julia> @code_typed pos(1)
    CodeInfo(
        1 _ %1 = Base.slt_int(x, 0)::Bool
        ___      goto #3 if not %1
        2 _      return 0
        3 _      return x
    ) => Int64
```
<br><br/>
Julia uses the LLVM compiler framework to generate machine code. LLVM
defines an assembly-like language which it uses as a shared intermediate
representation (IR) between different compiler optimization passes and
other tools in the framework. Julia uses LLVM's C++ API to construct the
LLVM IR in memory and then call some LLVM optimization passes on that
form. When you call \@code\_llvm you see the LLVM IR after generation
and some high-level optimizations of the method.

```julia
    julia> @code_llvm pos(1)

    ;  @REPL[1]:1 within `pos'
    define i64 @julia_pos_17096(i64) {
    top:
      %1 = icmp sgt i64 %0, 0
      %spec.select = select i1 %1, i64 %0, i64 0
      ret i64 %spec.select
    }
```
<br><br/>
Since Julia executes native code, the last form a method implementation
takes is what the machine actually executes. This is just binary code in
memory, which is rather hard to read. The \"assembly language\" form of
the method represents the instructions and registers with names and some
form of simple syntax to help express what instructions do. In general,
assembly language remains fairly close to one-to-one correspondence with
machine code. In particular, one can always \"disassemble\" assembly
code into machine code.

```julia
    julia> @code_native pos(1)
    .text
    ; _ @ REPL[1]:1 within `pos'
        movq    %rdi, %rax
        sarq    $63, %rax
        andnq   %rdi, %rax, %rax
        retq
        nopl    (%rax)
    ; _
```
<br><br/>
We mentioned that in the type inferred step, the compiler presents a
method implementation for a particular set of arguments. Let us take an
example to highlight this. The entire process starts when the function
*pos(x)* is called with say, an integer argument x = 1. The JIT compiler
now knows the type of x (which will be inferred in the *typed inferring
stage* of the compilation process). Using this information, it can then
compile a specialised version of the pos(x) to handle integers. This is
stored in the memory.

Suppose that the function *pos(x)* is called again, however this time
the argument passed as x, is of the type float. The JIT compiler will
again infer the types of the variables wherever necessary in the
function and generate a new specialised version to handle this type of
argument.

This entire feature is known as multiple dispatch. It can be perceived
that multiple dispatch is similar to static function overloading, found
in other languages but here the operation happens at runtime.

```julia
        julia> @code_typed pos(1)
        CodeInfo(
        1 ─ %1 = Base.slt_int(x, 0)::Bool
        └──      goto #3 if not %1
        2 ─      return 0
        3 ─      return x
        ) => Int64
```

```julia
        julia> @code_typed pos(1.2)
        CodeInfo(
        1 ─ %1 = Base.lt_float(x, 0.0)::Bool
        └──      goto #3 if not %1
        2 ─      return 0.0
        3 ─      return x
        ) => Float64
```

----------------
##### Type Instability
----------------
Let us declare a modified test method *pos1(x)* which will return the
value zero in integer type rather than in the type of x in the original
function, given that its return condition is fulfilled.

```julia
    julia> pos1(x) = x < 0 ? 0 : x
```
<br><br/>
We can see that for an integer input x, the typed form of code is
exactly the same. However, the code complexity steeply increases for a
input x which is of floating value type.

The reason for this is that zero is an integer (Int64 more specifically)
and x might be of any type. Thus depending one the value of x, this
modified method might return a value of either type, highlighted by
*UnionFloat64, Int64* in the generated code. This is one example of
type-instability, which can lead to slow down of code execution. One of
the reasons being that the compiler has to execute more number of
instructions to complete the same task (compare this typed pos1(x)
generated code with that of the code generated by pos(x)).

```julia
        julia> @code_typed pos1(1)
        CodeInfo(
        1 ─ %1 = Base.slt_int(x, 0)::Bool
        └──      goto #3 if not %1
        2 ─      return 0
        3 ─      return x
        ) => Int64
```

```julia
        julia> @code_typed pos1(1.2)
        CodeInfo(
        1 ─ %1 = Base.sitofp(Float64, 0)::Float64
        │   %2 = Base.lt_float(x, %1)::Bool
        │   %3 = Base.eq_float(x, %1)::Bool
        │   %4 = Base.lt_float(%1, 
                9.223372036854776e18)::Bool
        │   %5 = Base.and_int(%3, %4)::Bool
        │   %6 = Base.fptosi(Int64, %1)::Int64
        │   %7 = Base.slt_int(%6, 0)::Bool
        │   %8 = Base.and_int(%5, %7)::Bool
        │   %9 = Base.or_int(%2, %8)::Bool
        └──      goto #3 if not %9
        2 ─      return 0
        3 ─      return x
        ) => Union{Float64, Int64}
```
<br><br/>
One may ask why Julia enforce type-stability. Among the
multiple reasons, one strikingly important one is that Julia is not a
statically typed language. These type-stability issues exist in some
measure in all dynamic languages. Not enforcing type-stability also
allows the existence of a rich Julia type environment, allowing the
inclusion of parametric and dispatch types.

Type-stability is much more important factor for Julia compared to
several other dynamic languages, and the reason for this was nicely
summarised by one of this language's main developer -

*\"Basically we can only produce good code, possibly among the best of
any code produced by JIT thanks to LLVM's optimization passes, but can
only do so for good julia code (i.e. if you follow performance tip). If
you fail to do that, or write in a pattern that's frequently seen in
R/python/JS, the performance will be much slower compare to other JIT
out there since the JIT for those languages has to deal with these code
so they implements a lot of speculative or profiling based optimizations
to get good performance.\"* - Yichao Yu [^2]

In other words, Julia is a bit of a double edged sword. Written properly
it can be as fast as statically typed languages with its compilation
process. Written in a manner without keeping in mind the few general
guidelines to the language, it can be deceptively slow to work with.

----------------
**References:**

[^1]: [A Brief History of Just-in-Time](https://dl.acm.org/doi/10.1145/857076.857077)
[^2]: [Notes On The Julia Compiler JIT VS Static](https://discourse.julialang.org/t/notes-on-the-julia-compiler-jit-vs-static/4275/2)