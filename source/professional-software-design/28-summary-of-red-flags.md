---
title: Summary of Red Flags
order: 28
startLine: 5461
endLineExclusive: 5494
---

## Summary of Red Flags

Here are a few of of the most important red flags discussed in this book. The presence of any of these symptoms in a system suggests that there is a problem with the system’s design: Shallow Module: the interface for a class or method isn’t much simpler than its implementation (see pp. 25, 110).

Information Leakage: a design decision is reflected in multiple modules (see p. 31).

Temporal Decomposition: the code structure is based on the order in which operations are executed, not on information hiding (see p. 32).

Overexposure: An API forces callers to be aware of rarely used features in order to use commonly used features (see p. 36).

Pass-Through Method: a method does almost nothing except pass its arguments to another method with a similar signature (see p. 46).

Repetition: a nontrivial piece of code is repeated over and over (see p. 62).

Special-General Mixture: special-purpose code is not cleanly separated from general purpose code (see p. 65).

Conjoined Methods: two methods have so many dependencies that its hard to understand the implementation of one without understanding the implementation of the other (see p. 72).

Comment Repeats Code: all of the information in a comment is immediately obvious from the code next to the comment (see p. 104).

Implementation Documentation Contaminates Interface: an interface comment describes implementation details not needed by users of the thing being documented (see p. 114).

Vague Name: the name of a variable or method is so imprecise that it doesn’t convey much useful information (see p. 123).

Hard to Pick Name: it is difficult to come up with a precise and intuitive name for an entity (see p. 125).

Hard to Describe: in order to be complete, the documentation for a variable or method must be long. (see p. 131).

Nonobvious Code: the behavior or meaning of a piece of code cannot be understood easily. (see p. 148).
