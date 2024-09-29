% Seminar Winter 24/25: Fuzz Testing in Go
% Martin Sulzmann

# Content

* Registration and topic selection

* Fuzz testing overview

* Fuzzing in Go by example

* Seminar topics

* Meetings

* Participants

* FAQ

* References

# Registration and topic selection

Working language (report, talk) either German or English.

## Registration (first come first served)

Registration based on `first come first served' via intranet.


## Topic selection

Seminar topics (see below) are fixed. But there is some design space.

In our first meeting on Wednesday 13:15-13:45 via [zoom](https://h-ka-de.zoom-x.de/j/4837536496?pwd=dnlrTmVhWXlYOTFNMEhnYVNtRTJwZz09) I will give a short
intro to fuzz testing in Go. This will help to choose the topic that suits you best.


# Fuzz testing overview

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

In this seminar, we focus on dynamic analysis methods.
Our specific focus is on  *fuzzing* (aka [fuzz testing](https://en.wikipedia.org/wiki/Fuzzing)) methods that aim
to reduces the number of false negatives that may result
from dynamic analysis methods.
The idea behind fuzzing is to find inputs such that some faulty
program paths/parts are reached.

## The basic idea behind fuzzing

1. Start with some input value.

2. Run the program and monitor the program for abnormal behavior.

3. Adjust the input value and repeat step 2 until we run into a bug.

Sounds simple but there are many parameters to consider.

Finding input values:

* Roll the dice and take your chances.

* Start with some initial input seed based on which new generation of inputs are generated.

Ensuring inputs satisfy some structure:

* Suppose the input is a string.

* Strings must satisfy certain patterns (e.g. expressible in terms of regular expressions).

* How to effectively generate inputs in such a case?

Exploiting program structure:

* Black-box fuzzing. Nothing is known about the program.

* White-box fuzzing. We have full access to the source code and run-time.

* Grey-box fuzzing. Anything in between. For example, we are able to instrument the byte code and/or manipulate the run-time.


# Fuzzing in Go by example

Specifically, we consider fuzzing in Go
as it supports a built-in [fuzzer](https://go.dev/doc/security/fuzz/) since Go 1.18.


Let's take a look at fuzzing via some Go examples.

## Even

The following function is meant to return true if the number is even.
Otherwise, we return false.

~~~{.go}
func Even(i int) bool {
	if i > 100 {
		return false
	}
	if i%2 == 0 {
		return true
	}

	return false

}
~~~~~~

There is an obvious bug (for numbers above 100).
We may miss this bug if we only consider a limited number of test inputs
(that are all below 100).

As we can see below, thanks to fuzzing we can quickly locate the bug.


### Sources

Put the following files in a separate folder.

even.go:

~~~{.go}
package main

import "fmt"

func Even(i int) bool {
	if i > 100 {
		return false
	}
	if i%2 == 0 {
		return true
	}

	return false

}

func main() {

	fmt.Printf("%d => %t", 5, Even(5))

	fmt.Printf("%d => %t", 0, Even(0))

}
~~~~~~~

even_test.go:

~~~{.go}
package main

import "testing"


type testPair struct {
	input    int
	expected bool
}

func TestEven(t *testing.T) {

	testcases := []testPair{
		{5, false}, {0, true}, {50, true}}

	for _, tc := range testcases {
		res := Even(tc.input)
		if res != tc.expected {
			t.Errorf("isEven: %d => %t, want %t", tc.input, res, tc.expected)
		}

	}

}


func FuzzEvent(f *testing.F) {
	testinputs := []int{5,0,50}

    for _, tc := range testinputs {
        f.Add(tc)  // Use f.Add to provide a seed corpus
    }
    f.Fuzz(func(t *testing.T, in int) {
        res := Even(in)
        res2 := Even(in+1)
			if res == res2 {
			t.Errorf("Fail: %d => %t, %d => %t", in, res, in+1, res2)
        }
    })
}
~~~~~~~

Sample runs.

~~~
> go mod init example/even
>
> go test . --fuzz=Fuzz
fuzz: elapsed: 0s, gathering baseline coverage: 0/4 completed
failure while testing seed corpus entry: FuzzEvent/b61b643957dc4b40ef87b96e142889311e42671d0964206e68b63839d9ff72cf
fuzz: elapsed: 0s, gathering baseline coverage: 3/4 completed
--- FAIL: FuzzEvent (0.01s)
    --- FAIL: FuzzEvent (0.00s)
        even_test.go:37: Fail: 104 => false, 105 => false

FAIL
exit status 1
FAIL	example/even	0.247s
~~~~~~~



## Channel

Here's another example that makes use of channel-based concurrency.
The program below contains a deadlock.
The deadlock does not manifest itself for all program runs
but only under a specific schedule.

The deadlock is independent of the input value.
But it seems that if we wait long enough that the fuzzer eventually
uncovers the deadlock.

### Sources

Put the following files in a separate folder.

channel.go:

~~~{.go}
package main

// Deadlock possible!
func Runner(i int) {

	ch := make(chan int)

	go func() {
		ch <- i
	}()

	go func() {
		<-ch
	}()

	ch <- i

}

func main() {

	for i := 0; i < 50; i++ {
		Runner(i)
	}

}
~~~~~~~

channel_test.go:

~~~{.go}
package main

import "testing"

func FuzzRunner(f *testing.F) {
	testinputs := []int{5}

	for _, tc := range testinputs {
		f.Add(tc) // Use f.Add to provide a seed corpus
	}
	f.Fuzz(func(t *testing.T, in int) {
		Runner(in)

	})
}
~~~~~~~

Sample runs.

~~~~
> go mod init example/channel
>
> go run channel.go
>
> go test . --fuzz=Fuzz
fuzz: elapsed: 0s, gathering baseline coverage: 0/22 completed
fuzz: elapsed: 0s, gathering baseline coverage: 22/22 completed, now fuzzing with 10 workers
fuzz: elapsed: 3s, execs: 41407 (13798/sec), new interesting: 1 (total: 23)
fuzz: elapsed: 6s, execs: 41407 (0/sec), new interesting: 1 (total: 23)
fuzz: elapsed: 9s, execs: 41407 (0/sec), new interesting: 1 (total: 23)
fuzz: elapsed: 11s, execs: 41980 (350/sec), new interesting: 1 (total: 23)
--- FAIL: FuzzRunner (10.64s)
    fuzzing process hung or terminated unexpectedly: exit status 2
    Failing input written to testdata/fuzz/FuzzRunner/a4d1cc446bc342b8e28355319c67fde7264e5b2c398f91977758641d6ee4ca54
    To re-run:
    go test -run=FuzzRunner/a4d1cc446bc342b8e28355319c67fde7264e5b2c398f91977758641d6ee4ca54
FAIL
exit status 1
FAIL	example/channel	10.875s
~~~~~~~


Comment.
The fuzzer does not explicitly issue that there is a deadlock (all threads are blocked). But "hung" is a likely indication for a deadlock.

## Race

### Sources

Put the following files in a separate folder.

race.go:

~~~{.go}
package main

import "fmt"

// import "time"
import "sync"

// Data race possible
func Race(i int) {
	var m sync.Mutex
	x := 1

	go func() {
		m.Lock()
		x = 2
		m.Unlock()
	}()

	//	time.Sleep(time.Second)
	x = 3
	m.Lock()
	fmt.Printf("\n%d", x)
	m.Unlock()

}

func main() {

	for i := 0; i < 10; i++ {
		Race(i)
	}

}
~~~~~~~~

race_test.go:

~~~{.go}
package main

import "testing"

func FuzzRace(f *testing.F) {
	testinputs := []int{5}

	for _, tc := range testinputs {
		f.Add(tc) // Use f.Add to provide a seed corpus
	}
	f.Fuzz(func(t *testing.T, in int) {
		Race(in)

	})
}
~~~~~~~~


Sample runs.

~~~~
> go mod init example/race
>
> go run -race race.go
...
many, many times, no race warning is issued
~~~~~~

The reason why no race warning is issued is as follows.

1. The main thread starts and highly likely will run all code to completion

2. As part of the main thread, we create a new thread.

3. This thread might not get started at all (=> no data race)

4. If this thread gets started, it is highly that the thread only starts after the main thread has executed all its statement

5. As the (FastTrack style) data race predictor underlying go-race maintains the order among critical section, go-race is unable to detect the data race.

6. If we include the sleep statement, the main thread will run after the new thread. Then, go-race is able to detect the data race.

Let's try Go fuzzing.

~~~~
> go test . --fuzz=Fuzz -race
fuzz: elapsed: 0s, gathering baseline coverage: 0/8 completed
fuzz: elapsed: 0s, gathering baseline coverage: 8/8 completed, now fuzzing with 10 workers
fuzz: elapsed: 0s, execs: 4115 (14303/sec), new interesting: 1 (total: 9)
--- FAIL: FuzzRace (0.29s)
    --- FAIL: FuzzRace (0.01s)
        testing.go:1319: race detected during execution of test

    Failing input written to testdata/fuzz/FuzzRace/acdea50f284db86d7252b2b70923c57f7dd7486a517f5eb29619998b2a00d164
    To re-run:
    go test -run=FuzzRace/acdea50f284db86d7252b2b70923c57f7dd7486a517f5eb29619998b2a00d164
FAIL
exit status 1
FAIL	example/race	0.569s
~~~~~~~~~

Interesting. Data race detected! How?
Does Go fuzzing tap in the Go run-time and mutate thread scheduling?

Not sure. A more likely explanation (for detecting the data race) is that
the instrumentation done by Go fuzzing affects thread scheduling
and thus we are able to detect the data race.

## Summary

Fuzzing in Go seems well supported.

We can effectively uncover the bug in `Even`.

However, fuzzing in Go can only applied to test units of certain input type.
What if our test unit expects a custom type?

Fuzzing works for all Go programs including programs that make use of concurrency.
It seems that we can uncover the deadlock bug in `Runner` and the data race bug in `Race'. But these are simple examples.
In case of the channel example it took the fuzzer a long time (10secs+) till the Go fuzzer reports "hung". It is not clear if the Go fuzzing is aware of deadlocks. See the following [issue](https://github.com/golang/go/issues/48591).
It also remains to be seen how effective Go fuzzing is for concurrent programs where there are many more alternative schedules to consider.


# Seminar topics

Here comes a list of seminar topics.
Each topic is labeled as <b>Ti</b>.
It is possible to divide a topic into further subtopics in case
several students are interested in the same topic.

## Fuzzing in Go

For the following topics, we consider
Go's builtin [fuzzer](https://go.dev/doc/security/fuzz/).


### T1: Fuzzing in Go

1. Build your own "larger" Go application. For example, you could (re)program some of the `Softwareprojekt' exercises in Go.

2. Introduce some bugs

3. See how effective fuzzing is for bug finding

4. Report your experiences

### T2: How does Go fuzzing work

This topics investigates the fuzzing methods used behind Go's builtin fuzzer.

1. What kind of fuzzer is used in Go based on which principles?

2. Is it possible to extend the Go fuzzer? For example, to deal with custom data types, add your own mutation strategies, ...?

To get started some references:

[Go Fuzzing](https://go.dev/doc/security/fuzz/)

[Design Draft: First Class Fuzzing](https://github.com/golang/proposal/blob/master/design/draft-fuzzing.md)

[testing: add fuzz test support](https://github.com/golang/go/issues/44551)

It seems that the Go fuzzer is based on libfuzzer.
See [here](https://google.github.io/oss-fuzz/getting-started/new-project-guide/go-lang/#native-go-fuzzing-support)

## Fuzzing concurrent Go

We consider fuzzing to expose concurrency bugs in Go.
Besides Go's built-in fuzzer there exists several research tools
that are geared for fuzzing of concurrent Go applications.

### T3: Go's builtin fuzzer

How effective is [Go fuzzing](https://go.dev/doc/security/fuzz/) to detect concurrency bugs?

1. As a starting point, consider the bug scenarios and examples discussed in Autonome Systeme.

2. There are further fuzzing tools for concurrent Go. See below. Check out some of the examples used and.

3. Apply Go fuzzing and report your experiences.


### T4: GFuzz

GFuzz is a tool that adapts Go's run-time
system to mutates the order among channel communications.
The [GFuzz github repo](https://github.com/system-pclub/GFuzz) includes
the academic paper that explains GFuzz and also hosts the GFuzz tool.

1. Read the GFuzz paper

2. Install the GFuzz tool

3. Apply GFuzz on some examples (mentioned in the paper, your own, examples from Autonome Systeme, ...)

4. Report your experiences


### T5: GoPie

GoPie is another fuzzing tool to expose concurrency bugs in Go.
It is an improvement over GFuzz as more complex mutations of channel communications
are performed (by adapting Go's run-time system as done by GFuzz).

Here is the link to the [GoPie tool repo](https://github.com/CGCL-codes/GoPie)
The associated paper is the following: [Effective Concurrency Testing for Go via Directional Primitive-constrained Interleaving Exploration](https://chao-peng.github.io/publication/ase23/ase23.pdf)

1. Read the paper

2. Install the GoPie tool

3. Apply GoPie on some examples (mentioned in the paper, your own, examples from Autonome Systeme, ...)

4. Report your experiences


# Meetings

Our first meeting takes place on
Wednesday 13:15-13:45 via [zoom](https://h-ka-de.zoom-x.de/j/4837536496?pwd=dnlrTmVhWXlYOTFNMEhnYVNtRTJwZz09).

Details for all other meetings can be found below.

We meet in room E301 always Thursdays from 14:00 onward.
You need to attend at least three meetings in person.

## 10.10

Getting started

## 7.11

Discussion + presentations

## 21.11

Discussion + presentations

## 6.12

Discussion + presentations

## 12.12

Discussion + presentations

# Participants

TBA

Some advice.

1. Start a github repo (add description, examples, report, ...)

2. Attend our meetings and participate in discussions

3. After each meeting we collect a list of action items

4. Give 1-2 talks where you report about the progress you have made


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

# References

[Barton Miller's webpage](https://pages.cs.wisc.edu/~bart/fuzz/)

Warning. Lots of papers. You might get lost.

[fuzzing papers](https://github.com/wcventure/FuzzingPaper/tree/master)

Specific references.

[ConFuzz—A Concurrency Fuzzer](https://wcventure.github.io/FuzzingPaper/Paper/AISC19_ConFuzz.pdf)

Fuzzing for TSan for the purpose of data race prediction. No link to an implementation. Seems to require access to the source code (scalability issues for large projects?).

In most other languages, fuzzing does not seem to work out of the box.
For example, consider the following paper on
[A Usability Evaluation of AFL and libFuzzer with CS Students](https://dl.acm.org/doi/10.1145/3544548.3581178)

A recent paper on [Greybox Fuzzing for Concurrency Testing](https://www.comp.nus.edu.sg/~gregory/papers/asplos24.pdf) that seems interesting.
