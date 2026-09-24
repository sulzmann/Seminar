# Seminar Winter 26/27

## Seminar series on deadlock and data race prediction

Don’t let the title scare you: this seminar drops the heavy math in favor of a hands-on, engineering-first approach. You will get to act as a software auditor—deploying, testing, and breaking cutting-edge open-source tools from companies like Meta and Google to master real-world concurrency skills that AI tools cannot replicate.


## How this seminar works

This seminar is independent of the "other" seminar. We will meet regularly (about every 2-3 weeks).

There's no `final' presentation (20min + discussion) like in the "other" seminar. You will present your progress (incrementally) given a few short presentations. You can use github to host your seminar and use markdown for documentation. No formal seminar report (like at least 30 pages ...) is required.

The focus is on practice (some tool) but you might have to consult some paper for the technical details.

Below you find four seminar topics (T1-T4) covering deadlock and data race prediction.
The setup for each topic is about the same.
Each topic can be split up in case several students are interested in the same topic.

# Seminar topics

## T1: Chronos and RELAY — Scaling Static Data Race Detection to Go

Dynamic race detectors (like Go's built-in -race flag) only find bugs if your test suites happen to trigger the exact racy execution path at runtime. Static analysis aims to solve this by scanning all possible execution paths at compile-time without ever running the code. The classic [RELAY](https://cseweb.ucsd.edu/~lerner/papers/relay.pdf) paper laid the foundational architecture for how to scale static race detection to millions of lines of code. [Chronos](https://github.com/amit-davidson/Chronos) is an open-source tool that attempts to implement these heavy static analysis principles specifically for the Go programming language.

### The Mission

Act as an independent tool reviewer. You will deploy the open-source Go static analyzer Chronos, build a test suite to discover its boundaries, and use the structural concepts from the RELAY paper to analyze the core challenges of static code verification.

### Your Step-by-Step Task List

1. <b>Deploy the Engine:</b> Clone the open-source repository for [Chronos - A static race detector for the go language](https://github.com/amit-davidson/Chronos). Follow its setup guide and run the analyzer against a simple multi-threaded Go file that uses basic mutex locking.

2. <b>Stress-Test the Tool:</b> Chronos relies on tracking guarded memory accesses to find races. Write 3 minimal Go test programs to locate the architectural limits of the tool:

*  1 program using sync.Mutex that contains a clear data race (verify if Chronos catches it).

* 1 program using Go chan (channels) or sync.WaitGroup that contains a data race. *Hint: Look at Chronos's documentation regarding its known limitations with channel synchronization.*

3. <b>Deconstruct the RELAY Paper:</b> Read the RELAY paper. Skip the dense formal notation and focus heavily on how it builds "function summaries" to pass lock and memory access information up the call graph.

4. <b>The Evaluation Report:</b> Summarize your findings. Why do static tools like Chronos struggle with complex modern language features like Go channels, while a framework like RELAY could scale across massive legacy codebases? What are the inherent trade-offs between false positives and false negatives in static analysis?


## T2: Frama-C's RacerF — Lightweight Static Data Race Detection

Static analysis for C code is notoriously difficult due to complex pointer arithmetic and manual memory management. Most academic tools run into massive scalability issues when processing large programs. Published at ECOOP 2025, <b>RacerF</b> is a cutting-edge, lightweight plugin for the famous Frama-C analysis platform designed to bypass these limitations and catch data races quickly.

### The Mission

Act as a software security auditor. You will install the Frama-C environment, deploy the open-source RacerF toolchain, write a custom suite of multi-threaded C programs to test its boundaries, and analyze whether its "lightweight" design could be ported over to the Go ecosystem.

### Your Step-by-Step Task List

1. <b>Deploy the Analyzer:</b> Clone the open-source tool from the [Deadlock and Racer repository](https://github.com/TDacik/Deadlock_and_Racer). Set up the Frama-C analysis environment and run the provided test cases to ensure the plugin executes properly.

2. <b>Build the C "Crash Dummies":</b> Write 4 small multi-threaded C programs using Pthreads:

* 2 programs containing tricky data races to see if the tool catches them (False Negatives check).

* 2 programs that are safe but use non-trivial lock interactions to see if you can trick the tool into generating fake warnings (False Positives check).

3. <b>Deconstruct the 2025 Paper:</b> Read the [RacerF ECOOP 2025 paper](https://drops.dagstuhl.de/storage/00lipics/lipics-vol333-ecoop2025/LIPIcs.ECOOP.2025.37/LIPIcs.ECOOP.2025.37.pdf). Skip largely the formal stuff and focus on the architectural mechanics: How does RacerF manage to remain "lightweight" compared to classic static detectors?

4. <b>The Go Adaptation Brainstorm:</b> Evaluate how this approach maps to your modern Go knowledge. Could a similar lightweight syntax-driven strategy be adopted for a Go static checker, or do Go's unique features (like channels and select statements) require a completely different approach?

## T3: CProver's Deadlock Checker — Testing the Limits of Static Lockset Analysis

Static deadlock detectors scan multi-threaded code to find cyclical lock dependencies (e.g., Thread A holds Lock 1 and wants Lock 2, while Thread B holds Lock 2 and wants Lock 1). To scale to millions of lines of Linux code, state-of-the-art tools use <b>thread-local locksets</b>. However, when threads are dynamically spawned and destroyed via pthread_create and pthread_join, these local sets can completely lose track of which locks are globally active.

### The Mission

Act as an adversarial software verification auditor. Your job is to compile and run the deadlock analysis branch of the famous <b>CProver (CBMC)</b> verification suite. You will feed it a specific, highly deceptive C/Pthreads program designed by Prof. Sulzmann to see if you can trick an industry-grade analyzer into missing a blatant deadlock.

## Your Step-by-Step Task List

1. <b>Deploy the CProver Toolchain:</b> Clone the official cbmc repository and check out the specialized deadlock-analysis branch. Compile the suite following the codebase instructions.

2. <b>Compile the Test-Bed:</b> Instead of using standard gcc, compile your C test files using CProver's specialized compiler wrapper:

bash:

~~~
goto-cc my_deadlock_test.c -o my_deadlock_test.o
~~~~

Then analyze the compiled binary for hidden deadlocks using the instrumenter tool:

bash:

~~~
goto-instrument my_deadlock_test.o --show-deadlocks
~~~~

3. </b>The Fork/Join Trap (The Core Experiment):</b> Implement the exact C/Pthreads example provided in this seminar description (where a main thread acquires a lock, spawns a child thread, joins it, and then releases the lock). Run the CProver checker against it. Does the tool flag the potential deadlock, or does it suffer a False Negative because parent-thread locks are invisible inside child execution contexts?

4. </b> Deconstruct the ASE Paper:</b> Read the paper [Sound Static Deadlock Analysis for C/Pthreads (Kroening et al.)](https://arxiv.org/pdf/1607.06927). Focus entirely on how they define their thread-sensitive context framework. Use their own methodology section to explain why their tool succeeded or failed to track the global state in your fork/join test case.

### Some sample program to get started

~~~{.c}
#include <pthread.h>
pthread_mutex_t l1 = PTHREAD_MUTEX_INITIALIZER;
pthread_mutex_t l2 = PTHREAD_MUTEX_INITIALIZER;
void* thread_l1_l2(void*) {
pthread_mutex_lock(&l1); // acquire l1
pthread_mutex_lock(&l2); // wants l2 -> potential deadlock here
pthread_mutex_unlock(&l2);
pthread_mutex_unlock(&l1);
return NULL;
}
void* thread_l1(void*) {
pthread_mutex_lock(&l1); // wants l1 -> potential deadlock here
pthread_mutex_unlock(&l1);
return NULL;
}
int main() {
pthread_t tid1, tid2;
pthread_create(&tid2, 0, thread_l1_l2, 0);
pthread_mutex_lock(&l2); // acquire l2
pthread_t tid3; pthread_create(&tid3, 0, thread_l1, 0);
pthread_join(tid3, 0); // waits for thread that wants l1
pthread_mutex_unlock(&l2);
}
~~~~~~~

### References

Some [older link](https://www.cprover.org/deadlock-detection/) that refers a link to some package (likely finding further examples).

## T4: Vector Clocks vs. HB Sets — Inside the Python "Grace" Engine

Google's industry-standard Go race detector relies heavily on <b>Vector Clocks</b> implemented inside ThreadSanitizer (TSan). However, tracking vector clocks across massive parallel threads can become computationally complex. In their overlooked paper "Ready, set, Go! Data-race detection and the Go language," Daniel Fava and Martin Steffen proved that you could build a complete race detector using alternative <b>Happens-Before (HB) sets</b> instead of classic clocks. They proved this by routing raw TSan execution events into a pure Python verification engine.

## The Mission

Act as an execution engine analyst. You will deploy Daniel Fava's open-source [Grace race detector](https://github.com/dfava/grace). Because the entire state machine is written in a single, clean Python script (grace.py), your goal is to inject telemetry code to visualize exactly how memory sets evolve during parallel execution, comparing it directly against Go's native clock-based -race detector.

## Your Step-by-Step Task List

1. <b>Deploy and Telemeter the Sandbox:</b> Clone the open-source repository for the Grace Engine. Open src/grace.py and locate the core logic classes: HB, Var, Proc, and Chan. Inject simple print tracking loops inside the write(), read(), send(), and recv() methods to dump the internal sets onto your terminal.

2. <b>Execute the Race Showdown:</b> Create 3 small Go programs with multi-threaded bugs:

    * Program A: A basic data race on an unprotected variable.

    * Program B: A race hidden across asymmetric channel communication.

    * Program C: A completely thread-safe program with highly interleaved schedules.Run them first using Go’s native dynamic detector (go run -race). Then, feed their execution logs into your telemetered Python grace.py engine.

3. <b>Map the Vector Clock Disconnect:</b> Read the paper [Ready, set, Go! Data-race detection and the Go language](https://arxiv.org/pdf/1910.12643). Focus on the core formalisms detailing why tracking set intersections can substitute vector clocks. Using your injected print statements, illustrate how the Proc and Var sets expand and garbage-collect (gc()) dynamically to prevent memory explosions.

4. <b>The Research Evaluation:</b> Why has set-based race detection been largely overlooked compared to standard vector clocks? Based on your live test data, what are the primary performance advantages or scaling bottlenecks of keeping a literal set of events rather than a compact array of clock ticks?
