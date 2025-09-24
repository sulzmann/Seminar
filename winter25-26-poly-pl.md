# Seminar Winter 25/26: What you always wanted to know about programming (languages)

# Content


* Registration, seminar format and topic selection

* What you always wanted to know about programming (languages)

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

A practice topic likely involves some programming task. For example, implement
some tasks in programming language A and B and compare the features used to achieve the task.
Of course, a theory topic may involve some programming and a practice topic requires you to do some background reading.

Seminar topics (see below) are fixed. But there is some design space.
We can fine tune the topic based on your background.


# What you always wanted to know about programming (languages)


This seminar is about programming and programming languages. Is this still relevant? I can simply ask ChatGPT/some AI tool to solve a programming task?

Well, ChatGPT/some AI tools are not replacing programmers. AI tools empower programmers to become more productive. Good programming skills and a solid understanding of programming languages is therefore more important than ever.

Our specific focus here is on <b>polymorphism</b> in programming languages. Polymorphism refers to the possibility that a function or data structure can accommodate data of different types. There are various kinds of polymorphism as discussed [here](https://sulzmann.github.io/SoftwareProjekt/lec-cpp-advanced-poly.html). The most well-known kinds are subtype polymorphism and parametric polymorphism (aka "generics").

Some programming languages support various kinds of polymorphism. For example, Java supports subtype and parametric polymorphism. C++ supports subtype and ad-hoc polymorphism (aka "overloading"). Instead of parametric polymorphism C++ offers templates. 

Within each kind of polymorphism there may be variations. For example, Java and C++ support nominal subtype polymorphism whereas Go supports semantic subtype polymorphism.

The same kind of polymorphism may be called differently across programming languages. For example, Haskell type classes provide for a rich form of ad-hoc polymorphism. In Rust type classes are called traits and in Swift they are called protocols.

The above shows that polymorphism is everywhere. Check out the topics below and see if you find something that interests you.

# Seminar topics

Here comes a list of seminar topics.
Each topic is labeled as <b>Ti</b>.
It is possible to divide a topic into further subtopics in case
several students are interested in the same topic.

The first four seminar topics are linked to an academic paper.
The focus is more on theory. Read the paper and tell us what you think about the paper.
The remaining three topics cover a more general area (not linked to a single paper).

## T1: [On the State of Coherence in the Land of Type Classes](https://www.arxiv.org/pdf/2502.20546)

Covers Haskell, Rust, Scala and Swift.
You should have some familiarity with at least one of these languages.You should have

## T2: [An Interactive Debugger for Rust Trait Errors](https://arxiv.org/pdf/2504.18704)

Some background on [Rust](https://sulzmann.github.io/SoftwareProjekt/lec-rust.html)

## T3: [RUSTASSISTANT: Using LLMs to Fix Compilation Errors in Rust Code](https://dl.acm.org/doi/pdf/10.1109/ICSE55347.2025.00022)

The pdf should be publicly available. Within HKA vpn you should have access to acm digital library. Let me know if there are issues.

## T4:  [GenC2Rust: Towards Generating Generic Rust Code from C](https://dl.acm.org/doi/10.1109/ICSE55347.2025.00127)

The pdf should be publicly available. Within HKA vpn you should have access to acm digital library. Let me know if there are issues.

## T5: Generic Go

1. Familiarize yourself with Go.
2. The current compiler Go 1.24 has some limitations.
3. Come up with your own examples that highlight these limitations.
4. Develop some workarounds for these limitations.
5. Review existing Go projects that make use of generics. Do the current limitations of Go 1.24. have any impact?
6. Summarize your findings.

References.

[Go basics](https://sulzmann.github.io/ModelBasedSW/mbsw-go-basics.html)
[Go methods, interfaces, generics](https://sulzmann.github.io/ModelBasedSW/mbsw-go-advanced.html)
[Further examples](https://sulzmann.github.io/ModelBasedSW/mbsw-go-details.html)
[Generic Go limitations](https://sulzmann.github.io/ModelBasedSW/mbsw-go-generics-limit.html)

For 4. Use some AI tool to summarize known workarounds.

For 5. There exists a tool that allows you to automatically scan Github projects and summarize their use of generics. More details will be provided.


## T6: Expressive types

The concept of expressive typing features such as polymorphism allows us to type more programs in a safe way (without having to rely on unsafe features such as type casts etc).
Expressive types can also be used to reject more programs.

A typical example is the manipulation of values that are measured using different physical dimensions such as meter and feet. We clearly should not add a value measured in feet to a value measured in meters. There are cases where this happened and led to millions of euros lost!

[Here](https://go.dev/play/p/G1LdeIc8OwM) is a simple Go program that exploits parametric polymorphism to ensure that physical units match when adding two numbers.

1. Pick a programming language of your choice (one that provides rich forms of polymorphism).
2. Explore the use of expressive types by coming up with your own example.
3. Review existing projects (for example on Github) that exploit expressive types
4. Summarize your findings.

Further references.

[Expressive types in Go, Haskell, C++ and Agda](https://sulzmann.github.io/ModelBasedSW/mbsw-go-types.html).

Go [Generic interfaces](https://go.dev/blog/generic-interfaces) can also be exploited to express richer (type-based) program properties.

Use some AI tool for a literature review on the subject.


## T7: Type classes (and traits, protocols) are not types

Go and Java support interfaces. Interfaces (aka abstract classes) are collection of methods and represent types.

Languages such as Haskell, Rust and Swift do neither support OO classes nor interfaces.
Haskell supports type classes, Rust supports traits and Swift supports protocols.
Largely, all three features are equivalent.

But this fact is not clear when going through the Rust and Swift documentation.
Via some AI tools you can find lots of references (mostly talks) that discuss type classes,
traits and protocols. Discussions are mostly superficial (sketch examples, no rigorous arguments ...).


1. Work out some concrete examples that say connect traits to type classes.

For example, consider the following simple comparison of [Rust versus Haskell](https://sulzmann.github.io/ProgrammingParadigms/pp-haskell-vs-rust.html). You could do something similar for Rust and Swift.

2. Take a look at the (dictionary-passing) translation method behind type classes (traits and protocols). As a start, here are some lecture notes that explain the dictionary-passing translation by example.

[Haskell notes](https://sulzmann.github.io/ProgrammingParadigms/pp-haskell.html) on page 9.

[Rust notes](https://sulzmann.github.io/ModelBasedSW/lec-rust.html) on page 12.


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
