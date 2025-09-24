
# P1: Verwendung von Concurrency in Go

Art: Fallstudie

Vorbedingung: Gute Programmierkenntnisse. Grundlagen in Go (siehe Autonome Systeme).

Ziel:

- Welche und wievele Go Programme verwenden Concurrency?

- In welcher Form werden Generics verwendet?

- Z.B. gibt es Programme welche nur channels verwenden? Wie oft wird select verwendet? Werden Mutexe oft in Kombination mit channels verwendet?
Was ist mit Waitgroups etc?

Tool support ist vorhanden um Go Programme automatisch auf ihre Verwendung von Concurrency Primitive zu untersuchen.

FURTHER DETAILS

There are hardly any prior works.
The exception seems to be the following
thesis work on [AN EMPIRICAL STUDY OF CONCURRENT FEATURE USAGE IN GO](https://thescholarship.ecu.edu/server/api/core/bitstreams/ebb9c7d7-55e6-4880-8a5f-f76071af1d35/content)


How to measure "usage" of concurrency primitives?

There are two possible directions.

1. Apply some source code analysis (but even if channels and mutexes occur in the program text what's their interaction at run-time?)

2. Analyze execution traces


We consider option 2.
Some tool will be provided to obtain and analyze execution traces.



# P2: Verwendung von Generics in Go

Art: Fallstudie

Vorbedingung: Gute Programmierkenntnisse. Grundlagen in Go.

Hintergrund:

Seit Version 1.19 unterstützt die Go Programmiersprache Generics.
Im Vergleich zur Theorie ist die praktische Umsetzung von Generics in Go eingeschränkt.

Ziel:  Gegenstand der Arbeit ist die Untersuchung der Verwendung von Generics in Go und
       etwaiger Einschränkungen.

- Welche und wie viele Go-Programme verwenden Generics?

- In welcher Form werden Generics verwendet (z.B. generische Strukturen, ...)

- Welche Einschränkungen gibt es?

Dazu soll systematisch Github nach relevanten Go-Programmen gesucht werden.
Tool Support ist dazu vorhanden und wird zur Verfügung gestellt.

# P3: AdvocateGo- Dynamische Concurrency Analysen für Go

AdvocateGo ist ein unter Federführung der HKA/Prof. Sulzmann entwickeltes Tool zwecks dynamischer Concurrency Analysen für Go.

Ziel:
Evaluation und je nach Neigung Mitarbeit an der Erweiterung von AdvocateGo.

Gruppenarbeit ist möglich.

Vorgehen

1. Einarbeitung. Source finden sich hier
https://github.com/ErikKassubek/ADVOCATE

2. Evaluation anhand von Beispielen

3. Je nach Neigung weitere Evaluation, z.B. Mithilfe bei der Erstellung von Übungsaufgaben welche in Autonome Systeme verwendet werden, und/oder Mitarbeit an der Erweiterung von AdvocateGo

# P4: Smarter Regular Expression Matching - From Theory to Tools

Ever wondered how regexes really work under the hood? This project dives into the algorithms powering regex matching — from cutting-edge theory (derivatives, automata) to real-world tools (lexers, high-performance matchers). Combine theory and implementation, and push regex matching beyond what today’s libraries can do.

##### Description

Regular expressions are everywhere — from text search and data extraction to the heart of programming languages and compilers. But under the hood, regex matching involves subtle algorithmic trade-offs between efficiency, expressiveness, and implementability.

This project explores advanced techniques for regular expression matching and submatching, bridging theory and practice. Building on state-of-the-art approaches (e.g. partial derivatives for submatching, extended REs, and lexer generators), you will:

* Investigate algorithmic foundations of regex matching (determinization, derivatives, automata-based techniques).

* Design and implement prototype tools that go beyond "black-box regex libraries."

* Benchmark and experimentally evaluate competing approaches on realistic workloads.

Depending on your background and interest, directions may include:

* Implementing and evaluating submatching algorithms (e.g., partial derivatives, position automata, backtracking vs automata-based engines).

* Experimenting with extended regex features (intersection, complement, lookahead/lookbehind).

* Exploring state-of-the-art research in regex matching, such as linear-time matching, hybrid DFA/NFA approaches, or symbolic automata.

* Applying your work to practical domains, e.g. high-performance lexing, log analysis, or security applications (regex fuzzing, denial-of-service avoidance).

##### Skills Required

* Interest in both theory (algorithms, automata, complexity) and practice (implementing and evaluating prototypes).

* Basic background in formal languages and automata theory.

* Programming skills (functional or imperative, e.g. OCaml, Haskell, Rust, or C++).

##### What you’ll gain

* Hands-on experience at the interface of theory and systems.

* Understanding of how deep theory (partial derivatives, automata theory) impacts practical software tools.

* A project that could serve as a stepping stone toward advanced research in programming languages, verification, or compiler construction.

##### References / Starting Points

Cox, R. (2007). Regular Expression Matching Can Be Simple And Fast (but is slow in Java, Perl, PHP, Python, Ruby, ...)

Some of Prof. Sulzmann's papers:

* Regular expression sub-matching using partial derivatives

* A Flexible and Efficient ML Lexer Tool based on Extended Regular Expression Submatching



# P5: Tracer for Go events including non-atomic reads/writes based on ThreadSanitizer (TSan)

## Background

TSan is a C++ library to process acquire/release/write/read events for the purpose of data race prediction.
TSan is largely based on the FastTrack algorithm with some optimizations.

The Go run time maps concurrency primitives to acquire and release operations. These acquire and release operations are then processed by TSan for the computation of
happen before relations.

Non-atomic write/read operations
are instrumented at the LLVM level and when executing
the program, each write/read operation issues
a "TSan" call.

## Goals

We wish to record TSan events in a logfile for offline processing (e.g. for implementing our own data race predictors).

*This part is pretty much done and has been investigated in a series of student projects.*

* The latest version, some [Master Thesis](https://github.com/Proglang-Uni-Freiburg/TsanLogging)

* ...

* The first attempt, some [Bachelor Project](https://github.com/martinglauner/llvm-project)



We wish to integrate the TSan offline recorder into Go.

*This part yet needs to be done*.

Points to note.

* Go send/receive operations are mapped to sequences of acquire/release operations. At the moment, we only care about acquire and release operations (could be something for later how to enrich the trace so that the connection to the issuing send/receive can restored).

* Go also supports RWLock and therefore there are two kinds of "release" operations. Likely, the current TSan tracer maps both kinds to the same "release". This might need to be adjusted in the TSan tracer.



### Trace format

*We yet need to distinguish between the two kind of release operations.*

We wish to generate trace files in the following format

#### STD Format

STD (Standard) Format due to [rapid](https://github.com/umangm/rapid)

Recall that a trace is a sequence of events, each of which can be seen as a record e = <t, op(decor), loc>, where

* t is the identifier of the thread that performs e

* op(decor) represents the operation along with the operand. op can be one of r/w (read from/ write to a memory location), acq/rel (acquire/release of a lock object), fork/join (fork or join of a thread). The field decor is the corresponding memory location when op is r/w, lock identifier when op is acq/rel and thread identifier when op is fork/join .

* loc represents the program location corresponding to this event e.

The STD file format essentially represents a trace as a sequence of records, where the above three fields are separated by a |. A sample STD file looks something like:

~~~~
T0|r(V123)|345
T1|w(V234.23[0])|456
T0|fork(T2)|123
T2|acq(L34)|120
T2|rel(L34)|130
T0|join(T2)|134
~~~~


which represents a trace containing 6 events, performed by 3 different threads (T0,T1,T2). The 1st event is a read event on memory location V123 performed by thread T0. The 2nd event is a write on memory location V234.23[0] by thread T1. The 3rd event is performed by thread T0 and forks thread T2. The 4th event is an acquire event of lock L34 in thread T2. The 5th event release lock L34 in thread T2. The 6th event joins thread T2 in the parent thread T0.
