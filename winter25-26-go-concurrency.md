# Seminar Winter 25/26: Program Analysis with a focus on Concurrency and Go

# Content

* Registration, seminar format and topic selection

* Program analysis methods to detect bugs

* Seminar topics

* FAQ

# Registration, seminar format and topic selection

Working language (report, talk) either German or English.

## Registration (first come first served)

Registration based on `first come first served' via intranet.

## Seminar format

This seminar is independent of the "other" seminar. We will meet regularly (about every 2-3 weeks).

## Topic selection

Below you find a list of seminar topics.

Topics can either lean more towards theory or practice.

A theory topic is likely linked to an academic paper. The main task is to read the paper and give a summary of things you have learned from reading the paper.

A practice topic likely involves some tool + programming. For example,
experiment with the [Go race detector](https://go.dev/doc/articles/race_detector)
by using your own programs + other examples.
Of course, a theory topic may involve some tool + programming and a practice topic requires you to do some background reading.

Seminar topics (see below) are fixed. But there is some design space.
We can fine tune the topic based on your background.

# Program analysis methods to detect bugs

A software bug results from some coding error (part of the user program but the issue could also be part of the run-time system, ...).
When executing the software we may observe some faulty behavior.
This could be a fatal error (e.g. null-pointer dereferencing), some unexpected color scheme of the UI etc.

To analyze the behavior of programs there are two common methods.

*Dynamic analysis* (aka testing) methods consider a specific run
(e.g. obtained via a unit test) and analysis this program run
for some buggy behavior.
If the program run leads to bug then we are immediately done.
The advantage of dynamic analysis methods are that they
scale to large programs. The issue is we may face false negatives.
That is, we may miss a bug because there is no unit test that
leads to the execution of some faulty program part.

*Static analysis* methods build an abstraction of the program at compile-time and attempt to identify potential bugs based on this abstraction.
The advantage is that it is possible to explore all program paths/parts
without having to rely on some specific unit tests.
The downside of a static analysis is that it is prone to false positives.
To ensure that the analysis is decidable we may need to overapproximate the program's behavior.
We also need to track variables and references to these variables
which may lead to further imprecision of the analysis.
Another issue is that the exploration of all program paths is costly.
Hence, besides false positives, there is also a scalability issue
when facing larger programs.

There exist many refinements of both methods as well as hybrid methods
that use a dynamic analysis method to resolve potential false positives that
are reported by a static analysis.

Bugs are everywhere therefore it is important to know something about program analysis.
Check out the topics below and see if you find something that interests you.

# Seminar topics

Here comes a list of seminar topics.
Each topic is labeled as <b>Ti</b>.
It is possible to divide a topic into further subtopics in case
several students are interested in the same topic.

## T1: Classification of Go concurrency bugs

Besides classic bug scenarios such as data races
and deadlocks involving mutex operations, what other kind of concurrency bug patterns emerge in Go?
Consult the following references.

[Understanding Real-World Concurrency Bugs in Go](https://songlh.github.io/paper/go-study.pdf)

[GoBench: A Benchmark Suite of Real-World Go Concurrency Bugs](https://lujie.ac.cn/files/papers/GoBench.pdf)


1. Read the papers.
2. Summarize the various kinds of Go concurrency bugs.
3. What tools are available?
4. Write your own programs and play around with a selected number of tools.
5. If time permits, gather more Go concurrency bugs (especially newly reported, check out issues on github) to expand benchmarks, categorize new patterns.


## T2: Go dynamic data race prediction

Assumes knowledge of data race prediction methods, see [Autonome Systeme](https://github.com/sulzmann/AutonomeSysteme).

1. Check out [Go race detector](https://go.dev/doc/articles/race_detector)
2. Play around with your own examples
3. Consult [Dynamic data race prediction - TSan and examples](https://sulzmann.github.io/AutonomeSysteme/lec-data-race-examples.html)
4. go-race (aka TSan) has issues when dealing with a large number of (active) threads
5. Come up with your own examples.
6. Is this an issue in practice?

## T3: [DR.FIX: Automatically Fixing Data Races at Industry Scale](https://arxiv.org/pdf/2504.15637)

1. Read the paper.
2. Summarize the key ideas.
3. Come up with your own examples.
4. Potential issues with the approach? What about false positives and false negatives?

## T4: Goroutine leaks (aka partial deadlocks)

In Go there can be the situation that some but not all goroutines are blocked.
This is referred to as a partial deadlock.
Partial deadlocks are evil because the system is still running but the blocked goroutines
cause memory leaks.

The paper [Unveiling and Vanquishing Goroutine Leaks in Enterprise Microservices: A Dynamic Analysis Approach](https://arxiv.org/pdf/2312.12002)
discusses the issue in more detail.
There is also the [Goleak](https://pkg.go.dev/go.uber.org/goleak) to detect goroutine leaks.

1. Read the paper.
2. Play around with the goleak tool.
3. Come up with your own examples.
4. Summarize your findings.

## T5: Systematic Concurrency Testing for Go

The challenge for any dynamic analysis method is to find a schedule that exhibits the bug.
To tackle this challenge, the idea is to control the run-time scheduler
to expose the bug.

The [AdvocateGo](https://github.com/ErikKassubek/ADVOCATE) framework
supports various modes (fuzzers) to systematically explore the schedule among goroutines
and integrates earlier tools such as [GFuzz](https://github.com/system-pclub/GFuzz)
and [GoPie](https://github.com/CGCL-codes/GoPie)

1. Check out the AdvocateGo tool
2. Play around with some of the examples provided
3. Apply AdvocateGo on some larger examples. We will discuss which examples and which modes shall be applied
4. Report your experiences


## T6: [LOCKSMITH: Practical Static Race Detection for C](https://users.ics.forth.gr/~polyvios/toplas10.pdf)

Some classic work.

1. Read the paper
2. Summarize the key ideas.
3. What are the pros and cons of the approach? What about false positives and false negatives?
4. Could the approached be adopted to Go?

## T7: [RacerF: Lightweight Static Data Race Detection for C Code](https://drops.dagstuhl.de/storage/00lipics/lipics-vol333-ecoop2025/LIPIcs.ECOOP.2025.37/LIPIcs.ECOOP.2025.37.pdf)

More recent work.

1. Read the paper
2. Summarize the key ideas.
3. What are the pros and cons of the approach? What about false positives and false negatives?
4. Could the approached be adopted to Go?

The method is available as a [tool](https://github.com/TDacik/Deadlock_and_Racer).

## T8: [Sound Static Data Race Verification for C: Is the Race Lost?](https://dl.acm.org/doi/pdf/10.1145/3732933)

Another more recent work.

1. Read the paper
2. Summarize the key ideas.
3. Is the race really lost?

# FAQ

Q: What format do we need to use for the report?
A: It is fine to use markdown (e.g. Readme.md as part of your github repo). Ultimately, you will need to upload a pdf to the intranet. You can convert the markdown to pdf. If you wish you can also write a separate report (using latex, windows, ..., converted to pdf).

Q: How to get started?
A: Each topic comes with a description including some action items (links to tools, papers, ...).

Q: Can we use additional resources of what kind?
A: Do your own research. Try to stick to credible sources. Even blog articles are fine but do not believe everything that is said on the internet!

Q: Can we collaborate as some topics have a common focus?
A: Of course! Exchange examples, ideas, experiences, ... but try to write the report in your own words.

Q: Is it possible that several students work on the same topic?
A: Yes. Each topic can be divided into subtopics with a different focus. We will work out such things after students have registered and selected their topics.

Q: Will there be a `final' presentation (20min + discussion) like in the IWI seminar?
A: No. You are expected to present the progress you have made. But there is no fixed time limit nor do you need to waste time on the color arrangement of your slides :)
