# Finding the Right Layer

*An AI-assisted study of peephole optimization in the Solidity compiler, and what happened
when the patch met its reviewers.*

Term paper for **Blockchain Technologies 6115**, Information Systems Engineering,
Concordia University — December 2025, revised August 2026.

📄 [**Philippe_Laporte.pdf**](Philippe_Laporte.pdf)

## What this is

An attempt to add new peephole optimization patterns to the Solidity compiler using a
large language model as a research instrument, and an account of how the resulting patch
was received by the compiler's maintainers.

Three candidate patterns were generated. One was unsound and died in the semantic test
suite. One was sound but a bad bargain — it traded roughly 5,000 gas of deployment cost
for 6 gas on an error path correct programs never take. The third was real: the dispatcher
guard that checks whether a call carries at least four bytes of calldata can be written
`gt(calldatasize(), 3)` instead of `iszero(lt(calldatasize(), 4))`, removing one
instruction from essentially every contract the compiler emits.

That pattern was submitted as a peephole rule and **rejected in that form**. A maintainer
pointed out the same effect was available one layer up, in Yul code generation, as a
single line. That observation is the paper's central result: the agent was good at
recognizing a local rewrite and poor at judging which layer of a multi-stage compiler
should own it — and layer selection, not pattern discovery, was where the engineering
value lay.

## Result

Measured over a 22-contract corpus against `evmone`: **1 byte of runtime code and 3 gas
per external call**, uniformly, in every compilation unit that emits a dispatcher, and
nothing in the four that do not.

| | |
|---|---|
| Pull request | [argotorg/solidity#16315](https://github.com/argotorg/solidity/pull/16315) |
| Issue | [argotorg/solidity#16316](https://github.com/argotorg/solidity/issues/16316) |
| Status | Approved; awaiting results from the team's runtime gas benchmarking harness |
| Final change | One line in `libsolidity/codegen/ir/IRGenerator.cpp` |

## About the August 2026 revision

The original submission was written before the pull request was reviewed, so it defended
the peephole implementation that was subsequently dropped. The revision recasts the paper
around the outcome and corrects several things the original got wrong — a gas figure that
did not reproduce, four quotations attributed to the Solidity documentation that do not
appear in it, a claim that the compiler contains no gas cost model, and a termination
proof whose central premise the compiler's own source contradicts. Each correction is
marked in place rather than quietly fixed, because the pattern in how they arose is part
of what the paper is about.

*In memory of Laurie Hendren.*
