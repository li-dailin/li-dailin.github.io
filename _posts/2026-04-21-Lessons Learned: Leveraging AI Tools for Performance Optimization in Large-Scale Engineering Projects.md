---
title: 'Lessons Learned: Leveraging AI Tools for Performance Optimization in Large-Scale Engineering Projects'
date: 2026-04-21
permalink: /blogs/2026/lessons-learned-leveraging-ai-tools-for-performance-optimization-in-large-scale-engineering projects
excerpt:
  In this post, I share practical insights from a recent performance optimization effort on a large-scale engineering project. Rather than treating AI as an automated solver, I explore how integrating tools into the workflow—specifically for legacy code comprehension, benchmark scaffolding, and result analysis—can significantly accelerate the optimization cycle for complex codebases.
tags:
  - AI
  - 性能优化
---

## 1. Positioning AI in the Workflow

In a recent performance optimization effort for an engineering project, the types of problems encountered were actually quite diverse. They ranged from compute-bound numerical modules to I/O and infrastructure bottlenecks in data export and core pipelines. Many optimization opportunities aren't necessarily "unimaginable"; rather, the real hurdles are the slow pace of reading legacy code, the high cost of trial and error, and the long validation chains. The most direct help AI provided was accelerating these three specific aspects.

I didn't treat AI as an automated optimizer, but rather as a highly efficient auxiliary tool. It helped me read code, propose solutions, build testing scaffolds, and organize results. However, what ultimately made it into the final commit was strictly dictated by compilation, testing, and profiling data.

## 2. My Practical Workflow

Through iterative optimization attempts, I accumulated a common methodology. 

First, I use hotspot analysis, logs, and flame graphs to narrow down the problem scope as much as possible. Then, I provide the AI with a comprehensive prompt containing the module's purpose, platform constraints, compiler details, testing methods, permissible modification scope, and the observed errors or performance anomalies. Once I receive candidate solutions from the AI, I immediately run benchmarks, regression tests, and result comparisons to filter out invalid suggestions and retain the optimal solution.

In this workflow, AI doesn't exactly "give the right answer"; instead, *it rapidly generates a batch of verifiable candidate answers*.

## 3. Two Frequently Used AI Tool Categories

### 3.1 Code Assistants

These tools (Github Copilot/Antigravity/...) are essentially code assistants closely integrated with the IDE and agent workflows. They are best suited for *directly code-related tasks*. For example: how to partition parallel workloads, where to start refactoring the control flow of a legacy module, why a specific I/O segment is slow, the root cause of an error, or whether it's viable to prototype a small modification to test the waters. As long as the problem remains within their context window, they generally perform quite well.

For parallel optimizations in compute-heavy numerical routines, they can quickly *point out the viable granularity for parallelism*. They also proactively *highlight potential bottlenecks* like static states, shared buffers, or file I/O blocking. While their initial recommendations might not be drop-in ready, they are highly effective at identifying structural obstacles early on.

They are equally helpful for I/O-heavy issues in the underlying data formatting and file handling components. Problems like excessive system calls, inappropriate buffering strategies, or redundant overhead in read/write paths are notoriously slow to trace manually. Letting the AI list the suspects first, and then verifying them against the codebase, saves a massive amount of time.

Another highly practical use case for these tools is *building benchmarks and testing scaffolds*. 

To evaluate the impact of performance optimizations, we often have to do tedious but necessary work: batch-adding timers to scripts, auto-generating multi-threaded test commands, running pre- and post-optimization comparisons, aggregating per-task execution times, or quickly writing a script to visualize results. Previously, this was doable but consumed extra coding time. Now, I just ask the AI to generate a version matching the existing script style, tweak it slightly, and run it.

This has been a huge help. *Whether an optimization is worth pursuing rarely depends on "having the idea", but rather on "how fast you can get the data".* AI significantly accelerates this feedback loop.

Of course, these tools still have their shortcomings. They tend to confidently output a "seemingly parallelized" version that scales poorly in reality. Common pitfalls include excessively large critical sections, incomplete encapsulation of shared states, or leaving I/O on the hot path, resulting in code that runs but doesn't run fast. Therefore, it usually takes several iterations and phases to achieve satisfactory optimization results.

### 3.2 Analytical Tools

Simultaneously, I use chat models (Gemini/ChatGPT/...) as analytical assistants. Before diving into optimization, I feed the target code to the AI to analyze potential hotspots, bottlenecks, and the overall optimization headroom.

It excels at deconstructing a performance issue across multiple layers: memory access patterns, loop structures, vectorization feasibility, parallel potential, and platform constraints. For hotspot function analysis or answering "is this direction worth exploring?", its value lies not in pinpointing specific lines of code, but in establishing a robust analytical framework first.

This serves two practical purposes for me:
1.  **Prioritization:** Some optimizations look fancy but yield low returns; some issues masquerade as algorithmic bottlenecks when the real culprit is I/O or the testing methodology itself. Putting the problem into a broader framework prevents a lot of wasted effort.
2.  **Drafting Presentation Materials:** Explaining why a specific optimization direction was chosen, why a proposed solution was abandoned, or how to interpret the test results can be disjointed if organized ad-hoc. AI saves a lot of effort in structuring technical processes into cohesive, presentable narratives.

However, it also has its boundaries. It is generally more reliable at the analytical level than at the implementation level. Especially when dealing with runtime exceptions, cluster environment quirks, or system-level bottlenecks, blindly pasting errors to a chat model is less effective than using an agent. Ultimately, you still have to rely on logs, monitoring, and empirical results.

## 4. The Real Value Adds of AI

### 4.1 Faster Comprehension of Legacy Modules
The difficulty of massive legacy codebases lies not just in the core algorithms, but in their historical baggage, varying coding conventions, and poor readability. AI can dissect these modules upfront, showing me which parts are core computations, which are control flows, and which are just vestigial structures for legacy compatibility. This drastically accelerates the onboarding process for new modules.

### 4.2 Rapid Generation of Candidate Optimization Strategies
Whether it's parameter-space parallelism, in-memory I/O, or system call optimization, AI can provide several actionable strategies—ranging from conservative to aggressive—for the user to choose from. The primary value here isn't getting it right on the first try, but heavily compressing the time from "identifying the problem" to "testing the first prototype."

### 4.3 Automating Benchmarks and Validation Workflows
This is where I feel the most tangible effort reduction. Getting pre- and post-optimization comparisons usually requires writing auxiliary commands and scripts. AI is incredibly adept at this. For instance:
* Batch-wrapping existing Python scripts with `timer` logic to profile per-task or per-step latency.
* Auto-generating suites of benchmark commands to compare results across optimizations, thread counts, and input scales.
* Rapidly writing temporary scripts to extract the exact data we care about from system tests.
* Formatting the output into clean, intuitive structures.

While not technically complex, the speed at which these tasks are completed directly impacts the cadence of the optimization iterations. Thus, beyond improving the quality of ideas, AI fundamentally lowers the cost of validation.

### 4.4 Explaining Suboptimal Results
During optimization, many minor tweaks fail to yield the expected results. Here, AI's role isn't to provide a definitive conclusion, but to list suspicious areas and a troubleshooting hierarchy. I then combine this with logs and profiling data to verify, which is far more efficient.

### 4.5 Boosting Presentation Material Efficiency
Beyond the code itself, AI helps me rapidly synthesize results, distill lessons learned, and organize materials. This value is highly pragmatic: project optimization isn't just about writing code; it's about articulating the process, outcomes, and insights clearly. For scenarios like project syncs, AI is instrumental in translating technical labor into presentable content.

## 5. Key Takeaways

1.  **Validation is King:** AI fits perfectly into a performance optimization workflow, provided it is strictly positioned *upstream* of testing and validation. Divorced from benchmarks, regression tests, and result comparisons, many suggestions look reasonable but fail in practice.
2.  **Human Judgment Remains Essential:** AI's core strengths lie in reading code, proposing candidates, building test scaffolding, and structuring technical expression. The final architectural decisions—evaluating module structures, pinpointing true bottlenecks, and judging result credibility—still belong to the developer.
3.  **Context is Crucial:** The more specific the prompt, the more useful the output. Providing comprehensive information—module background, platform environment, compiler specs, input scale, acceptable modification scope, anomalies, and error logs—drastically improves AI performance.
4.  **Decouple Tools for Efficiency:** Separating the "Agent" from the "Plan" yields higher efficiency than forcing a single tool to do everything. Agents are closer to implementation and prototyping, while chat models are better suited for taking a step back to analyze and articulate.
5.  **Ownership of the Final Commit:** What ultimately gets committed is what compiles, passes tests, and can be clearly explained. AI can make you walk faster, but it cannot make the final judgment for you.

## 6. Conclusion

Overall, the value proposition of these AI tools in this project's performance optimization process is highly definitive. They cannot fully automate the optimization pipeline, but they tangibly accelerate the granular implementation details: reading code, brainstorming solutions, scaffolding benchmarks, executing validations, and synthesizing results. For an HPC engineering project characterized by legacy code, complex modularity, and high validation costs, this auxiliary value is profoundly real and practical.
