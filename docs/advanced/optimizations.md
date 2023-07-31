# Optimization Options

There are cases when using solely the `-Ox` options will not bring the desired optimization (either size or speed)
for a compiled function/application. In these cases we need to understand where is the program’s bottleneck and
if it can be solved either by passing various options to the compiler or by source code modifications. In this
section, we look into compiler’s command-line options and how they can help us in achieving better results.

## Architecture-Independent Optimizations

The first step in optimizing your code is by experimenting with architecture-independent optimizations. Almost any
GCC pass (i.e., optimization) can be turned on or off or steered using parameters. These optimizations are denoted
by the notation `-fxxxx`, where `xxxx` is the GCC pass that is turned on. To turn off a gcc pass, we need to pass
`-fno-xxxx` to the compiler. The same observation holds for other types of optimizations such as the architecture-specific
ones. For more information about GCC options, please check the [GCC manual][gcc-manual]. It is highly desirable to know
and understand how these options work in order to properly use them.

Here is a list of most reasonable options:

`-ftree-loop-ivcanon`

:   Create a canonical counter for number of iterations in loops for which determining number of iterations requires
    complicated analysis. Later optimizations then may determine the number easily. Useful especially in connection
    with unrolling.

`-ftree-vectorize`

:   Perform loop vectorization on trees. This flag is enabled by default at `-O3`. This option is useful to use either
    if the ARC processor does not have the SIMD extensions as it performs extra code analysis and may improve the
    following optimizations.

`-ftree-loop-if-convert`

:   Attempt to transform conditional jumps in the innermost loops to branch-less equivalents. The intent is to remove
    control-flow from the innermost loops in order to improve the ability of the vectorization pass to handle these
    loops. This is enabled by default if vectorization is enabled.

`-f[no-]tree-dominator-opts`

:   Perform a variety of simple scalar cleanups (constant/copy propagation, redundancy elimination, range propagation
    and expression simplification) based on a dominator tree traversal. This also performs jump threading (to reduce
    jumps to jumps). This flag is enabled by default at `-O` and higher.

`-f(no-)ivopts`

:   Perform induction variable optimizations (strength reduction, induction variable merging and induction variable
    elimination) on trees. Disabling the `ivopts` optimization may improve the number of hardware loops recognized by
    the compiler.

`-fselective-scheduling`

:   Schedule instructions using selective scheduling algorithm. Selective scheduling runs instead of the first
    scheduler pass.

`-fgcse`

:   Perform a global common subexpression elimination pass. This pass also performs global constant and copy propagation.
    It may be useful to disable this step specially when we want to have more `SUB1/2/3`, `ADD1/2/3` type of
    operations generated.

`-frename-registers`

:   Attempt to avoid false dependencies in scheduled code by making use of registers left over after register
    allocation. This optimization most benefits processors with lots of registers. Depending on the debug information
    format adopted by the target, however, it can make debugging impossible, since variables no longer stay in a
    "home register". Enabled by default with `-funroll-loops` and `-fpeel-loops`.

`-fira-loop-pressure`

:   Use IRA to evaluate register pressure in loops for decisions to move loop invariants. This option usually
    results in generation of faster and smaller code on machines with large register files (>= 32 registers),
    but it can slow the compiler down.

`-fsched-pressure`

:   Enable register pressure sensitive instruction scheduling before register allocation. This only makes sense when
    scheduling before register allocation is enabled, i.e. with `-fschedule-insns`. Usage of this option can improve
    the generated code and decrease its size by preventing register pressure increase above the number of available
    hard registers and subsequent spills in register allocation.

`-f[no-]regmove`

:   Attempt to reassign register numbers in move instructions and as operands of other simple instructions in order
    to maximize the amount of register tying. This is especially helpful on machines with two-operand instructions.
    Disabling this optimization may result in faster code.

## Optimizations for Code Size

If code size is our target, beside the GCC's `-Os` option, it may make sense to use it in conjunction with
following command-line options:

* `-fsection-anchors`
* `-fno-branch-count-reg`
* `-fira-loop-pressure`
* `-fira-region=all`
* `-fno-sched-spec-insn-heuristic`
* `-fno-move-loop-invariants`
* `-fno-tree-dominator-opts`
* `-ftree-vectorize`
* `-fno-cse-follow-jumps`
* `-fno-jump-tables`

## Optimization for Speed

If the cycle count is our target, the best is to start with `-O2` option then with `-O3` and for each compiler
optimization level to combine one or more of the suggested GCC command-line options. Finally, gather and compare
runtime performance and size for each command-line combination. Consider plotting these numbers on a 2D graph,
where one axis will represent the cycle count, and the other will represent the size. Hence, we can choose the best
combination size/speed for a given problem.

If one wants to try a large number of option combinations, then an automatic scripting process is required. One of
those tools that searches through more than 1.3 zillion gcc option combination is `Acovea`. `Acovea` is using genetic
algorithms to search for the best option combination for a given program. However, one can make an script that uses
only the suggested gcc options to search for the best combination by exhaustively generating (most) of the option
combinations.

## Using `optimize` attribute

In GNU C, you declare certain things about functions called in your program which help the compiler optimize
function calls and check your code more carefully. In the case when we want a certain function/kernel not to
change its speed/size characteristics, we can use the `optimize` function attribute. The `optimize` attribute
is used to specify that a function is to be compiled with different optimization options than specified on the
command line. Arguments can either be numbers or strings. Numbers are assumed to be an optimization level.
Strings that begin with O are assumed to be an optimization option, while other options are assumed to be used
with a `-f` prefix.

## Default ARC Optimizations

| Opitimization          | `-O0` | `-O1` | `-Os` | `-O2` | `-O3` |
|------------------------|-------|-------|-------|-------|-------|
| `-fomit-frame-pointer` |       | On    | On    | On    | On    |
| `-fschedule-insns`     |       | On    | On    | On    | On    |
| `-fschedule-insns2`    |       | On    | On    | On    | On    |
| `-mearly-cbranchsi`    | On    | On    | On    | On    | On    |
| `-mbbit-peephole`      | On    | On    | On    | On    | On    |
| `-mcase-vector-pcrel`  |       |       | On    |       |       |
| `-mcompact-casesi`     |       |       | On    |       |       |

## Default ARC Parameters

| Parameter                 | Value |
|---------------------------|-------|
| `simultaneous-prefetches` | 4     |
| `prefetch-latency`        | 4     |
| `l1-cache-line-size`      | 64    |

[gcc-manual]: <https://gcc.gnu.org/onlinedocs/gcc/Option-Summary.html#Option-Summary> "GCC manual"
