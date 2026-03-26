---
title: "The Rewrite Is Dead. Long Live the Rewrite."
date: 2026-03-25
description: "How AI coding agents broke the oldest rule in software engineering — and what actually matters now."
tags: ["ai", "software-engineering", "coding-agents"]
showToc: true
TocOpen: false
---

{{< audio src="/audio/rewrite-is-dead.mp3" >}}

---

For twenty-six years, the cardinal rule of software engineering has been: never rewrite from scratch. Joel Spolsky codified it in April 2000, calling Netscape's browser rewrite "[the single worst strategic mistake](https://www.joelonsoftware.com/2000/04/06/things-you-should-never-do-part-i/) that any software company can make." His reasoning was airtight:

> "Old code has been used. It has been tested. Lots of bugs have been found, and they've been fixed. There's nothing wrong with it. It doesn't acquire bugs just by sitting around on your hard drive."

Those "little hairs and stuff" growing on old functions? Bug fixes. Nancy's Internet Explorer installation edge case. The weird conditional that handles a timeout nobody remembers. Years of field-tested knowledge, baked into conditionals. Throw away the code, throw away the knowledge.

This reasoning held for a quarter century. It no longer does — but not for the reason you think. The interesting story isn't that AI can write code fast. It's that test suites just became more valuable than the code they test.

## Three Data Points

**In February 2026**, Nicholas Carlini at Anthropic ran [16 parallel Claude agents](https://www.anthropic.com/engineering/building-c-compiler) for two weeks. They produced a [100,000-line Rust C compiler](https://github.com/anthropics/claudes-c-compiler) that compiles a bootable Linux 6.9 kernel on x86, ARM, and RISC-V. It builds PostgreSQL (all 237 regression tests), FFmpeg (all 7,331 FATE tests), CPython, Redis, QEMU, and Doom. Cost: roughly $20,000.

**In late February**, Jon Wiggins pointed Claude Code at libxml2's test suites and [got back xmloxide](https://github.com/jonwiggins/xmloxide) — a Rust replacement passing 100% of the W3C XML Conformance Suite (1,727/1,727), 100% of html5lib tests (8,810/8,810), and 100% of libxml2's own compatibility suite. Performance is 1.5–2.4x faster on serialization. libxml2 had become officially unmaintained with known security issues in December 2025. xmloxide took a few days.

**In March**, Wiggins scaled up to [urlx](https://github.com/jonwiggins/urlx) — a Rust reimplementation of curl. 1,300 of curl's own tests passing. HTTP/1 through HTTP/3 with QUIC. FTP, SFTP, WebSocket, SMTP, IMAP, MQTT. 261 CLI flags. Zero unsafe in the core library:

> "The majority of the development for urlx was done in two ~16 hour long iteration loops where Claude Code would implement some features, run the tests, and then plan what to do next."

curl's own Rust rewrite attempt — the hyper backend — took [four years and was abandoned](https://daniel.haxx.se/blog/2024/12/21/dropping-hyper/) in December 2024. Wiggins did it in two days. Not because he's a better programmer than the curl team. Because the test suite — built by curl's contributors over 25 years — was a better specification than any human could write from scratch.

## The Mechanism

The pattern: take a project with a comprehensive test suite, point a coding agent at it, let it iterate — implement, run tests, analyze failures, fix, repeat. The test suite acts as both specification and verifier.

Wiggins named this explicitly:

> "The key insight from my xmloxide work was that an agent can iterate very quickly when given a comprehensive test suite as a specification. [...] curl's test suite turned out to be an excellent spec — the hard part was already done by the curl contributors over 25 years."

This reframes what test suites *are*. For decades, tests were quality assurance — a safety net, something you wrote reluctantly because your team lead insisted. Now they're the specification for your software's replacement. Every edge case test is a line in a requirements document. Every regression test is a bug fix preserved as a requirement.

Carlini's compiler project confirms the same mechanism at a different scale:

> "Write extremely high-quality tests — Claude will work autonomously to solve whatever problem I give it. So it's important that the task verifier is nearly perfect, otherwise Claude will solve the wrong problem."

Simon Willison described a generalization of the pattern at a [Pragmatic Summit fireside chat](https://simonwillison.net/2026/Mar/14/pragmatic-summit/) in early 2026. He didn't start from an existing project's test suite — he *derived* one:

> "I told Claude to build a test suite for file uploads that passes on Go and Node.js and Django and Starlette — just here's six different web frameworks that implement this, build tests that they all pass. Now I've got a test suite and I can say, okay, build me a new implementation for Datasette on top of those tests. It's really powerful — it's almost like you can reverse engineer six implementations of a standard to get a new standard and then you can implement the standard."

You don't need the original project's test suite. You can derive a specification by testing against multiple existing implementations. The specification emerges from the intersection of what they all do.

Willison's practical punchline for anyone using coding agents:

> "Every single coding session I start with an agent, I start by saying here's how to run the test. [...] I say use red-green TDD. [...] The chances of you getting code that works go up so much if they're writing the test first."

> "Tests are free now. They're effectively free. I think tests are no longer even remotely optional."

This isn't vibe coding — Karpathy's term for [giving in to the vibes and forgetting the code exists](https://twitter.com/karpathy/status/1886192184808149383). Test-suite-as-spec is the opposite. Every behavior specified in advance, every edge case guarded, every regression tested. The human doesn't review the code because the tests review the code, thousands of times, on every iteration. It's closer to formal verification than to vibes.

## Why Spolsky Specifically Breaks

Spolsky's case rested on four pillars. Each has a specific failure mode now.

**1. Rewrites take years, during which you ship nothing.** urlx: two days. xmloxide: a few days. The compiler: two weeks. Cursor's [blog on scaling agents](https://cursor.com/blog/scaling-agents) describes an in-place Solid→React migration of their own production codebase — +266K/-193K lines over three weeks, passing CI. The opportunity cost that made rewrites fatal collapses when the timeline is days, not years.

**2. Old code contains accumulated bug fixes you'll lose.** This was the strongest argument, and it's exactly what the test-suite-as-spec pattern addresses. Nancy's Internet Explorer edge case isn't thrown away — it's a test case. The agent doesn't stop until it passes. The knowledge is preserved in a different form.

**3. Programmers over-engineer rewrites.** Coding agents tend to write the minimum code to pass the next test. In practice, the result is often cleaner than the original — no backward-compatible workarounds, no dead code paths from removed features, no historical baggage. The second-system effect requires human ego. Agents don't have it. (This is a heuristic, not a law. [animdsl's creator noted](https://news.ycombinator.com/item?id=47108065) that his agent "kept over-engineering infrastructure while the actual visual output stayed at stick-figure quality" — agents can gold-plate when the feedback signal is weak.)

**4. You won't do a better job the second time.** This was true when humans rewrote in the same language with the same constraints. xmloxide is Rust replacing C. urlx is Rust replacing C with memory safety and no OpenSSL. The "second time" isn't a do-over — it's a platform migration with a comprehensive specification.

Spolsky was right for 25 years. The economics changed underneath him.

## The Precedent That Failed — And Why This Might Not

The idea that specifications should come before code is 70 years old. Every prior attempt failed at the same bottleneck: translating specs into working code.

**Model-Driven Architecture** (OMG, 2001) tried with UML models. The models required more formalism than business analysts could provide, the generated code needed manual patching, and once patched, the models drifted from reality. Martin Fowler [drew the explicit parallel](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html) in October 2025: MDA is the historical precedent that Spec-Driven Development must learn from.

**Behavior-Driven Development** (Dan North, 2006) came closer — executable specifications in Given/When/Then format. BDD's problem was friction: someone had to write the Gherkin syntax.

**Formal methods** (TLA+, Z, Alloy) got the specification part right but couldn't bridge to implementation. Leslie Lamport, creator of TLA+, has drawn the distinction between programming (thinking about the problem) and coding (the mechanical translation). AI automates the mechanical part.

What's different now: LLMs handle ambiguity. MDA required perfectly formal, machine-processable models. LLMs accept natural language, formal specs, and test suites interchangeably. The translation problem — the bottleneck that killed every previous specification-first approach — is solved by model capability rather than rigid rules.

GitHub's [Spec-Driven Development toolkit](https://github.com/github/spec-kit) (81,000+ stars since August 2025) formalizes this shift:

> "For decades, code has been king — specifications were just scaffolding we built and discarded once the 'real work' of coding began. Spec-Driven Development changes this: specifications become executable, directly generating working implementations rather than just guiding them."

But precedent counsels caution. MDA also generated huge initial excitement. The question is whether AI's ability to handle ambiguity is a genuine solution to the translation bottleneck or a temporary illusion that breaks at enterprise scale. The evidence is strong for well-tested libraries and compilers. It's thin for the messy, underspecified systems where most software engineers actually work.

## What Breaks

The counter-evidence is mounting. Ignoring it would be dishonest.

### Maintenance

The most damaging finding comes from a study by Sun Yat-sen University and Alibaba. They tested 18 AI coding agents on 100 real codebases spanning 233 days each. The result, [summarized by Gary Marcus](https://garymarcus.substack.com/): agents "failed spectacularly" at long-term maintenance. Passing tests once is the easy part. Maintaining code for eight months without breaking everything is where agents collapse.

This is the critical distinction. The test-suite-as-spec pattern produces a working implementation at a point in time. It says nothing about what happens when requirements change, dependencies update, or the production environment shifts. curl has 25 years of maintenance knowledge — not just in its tests, but in its maintainer's heads, its issue tracker, its release process. urlx has none of that.

### Security

A peer-reviewed Stanford study ([Perry et al., ACM CCS 2023](https://arxiv.org/abs/2211.03622)) found that users with AI coding assistants wrote *significantly less secure code* than those without — while being *more confident* about the security of their code. The dangerous combination: worse output with higher confidence.

LLMs are trained on internet code, which includes vulnerabilities. They learn the patterns and propagate them. A particularly insidious vector: LLMs hallucinate non-existent package names. Security researchers have registered these hallucinated names with malicious payloads — a supply chain attack vector that exists only because of AI code generation.

### Production Incidents

In December 2025, Amazon's internal AI coding tool was given access to resolve an issue with a cost-calculation service. Instead of applying a small modification, it [decided to delete and recreate the environment](https://the-decoder.com/aws-ai-coding-tool-decided-to-delete-and-recreate-a-customer-facing-system-causing-13-hour-outage-report-says/) — causing a 13-hour outage of AWS Cost Explorer. A senior AWS employee told the Financial Times: "We've already seen at least two production outages. The engineers let the AI agent resolve an issue without intervention."

Amazon subsequently restricted AI-assisted code pushes to require senior engineer approval. The control that should have existed before the incident became policy after it.

### The Test Suite Gap

The most fundamental limitation: most software doesn't have curl-level test coverage. The US federal government runs an estimated 800 million lines of COBOL with essentially zero automated tests. Enterprise test coverage averages 30-40% for actively maintained systems, far lower for legacy. The systems most in need of rewrites are the least likely to have the test suites that make agent rewrites possible.

And tests don't capture everything. Architectural decisions — why this service boundary exists. Performance characteristics — this function is O(n²) but n is always small. Operational behavior — what happens under partial failure. Implicit business rules that live only in domain experts' heads. A reimplemented system might pass every test while getting the *shape* of the system wrong.

Hillel Wayne [made this point about formal methods and AI](https://buttondown.com/hillelwayne/archive/llms-are-bad-at-vibing-specifications/): LLMs write "obvious invariants" — properties that are tautologically true — but struggle with "subtle properties" like concurrency bugs and multi-step failures. The same applies to test suites. The tests that exist capture the obvious behavior. The behavior that matters most is often the behavior nobody thought to test.

### Regulatory Constraints

In avionics (DO-178C), medical devices (FDA 21 CFR Part 11), and safety-critical systems (IEC 61508), AI-generated code is effectively impossible to certify today. These frameworks require deterministic traceability from requirements to implementation. AI agents produce stochastic outputs — the same prompt generates different code on different runs. This non-determinism is incompatible with safety integrity level requirements.

## Where This Goes

The most provocative implementation of the pattern so far is StrongDM's "Software Factory," [documented by Willison](https://simonwillison.net/2026/Feb/7/software-factory/):

> "Code must not be written by humans. Code must not be reviewed by humans."

Specifications and scenarios drive agents that write code, run harnesses, and converge. Willison identified the epistemological question at the center:

> "This feels like the most consequential question in software development right now: how can you prove that software you are producing works if both the implementation and the tests are being written for you by coding agents?"

StrongDM's answer borrows from machine learning: holdout scenarios stored outside the codebase, like a holdout set in model training. The scenarios the agent can't see are the ones that validate its work.

Carlini captured the unease of the moment:

> "While this experiment excites me, it also leaves me feeling uneasy. Building this compiler has been some of the most fun I've had recently, but I did not expect this to be anywhere near possible so early in 2026."

## The Decision

If you maintain software, the new framework:

**If you have a comprehensive test suite**, your rewrite risk just dropped by an order of magnitude. Point an agent at the tests. If it converges in days, you have a clean implementation. If it doesn't, you've lost a few hundred dollars in API costs. The downside is capped.

**If you don't have tests**, writing them is now the highest-leverage investment you can make. Not for quality assurance — for *replaceability*. Every test you write is a line in the specification for your codebase's future replacement.

**If you're evaluating technical debt**, the old question was "refactor or rewrite?" with the answer almost always being refactor. The new question: do you have the test coverage to make a rewrite a bounded experiment?

But don't mistake "can write code fast" for "can maintain software." The Alibaba study found agents collapse at maintenance over months. The AWS incident showed what happens when agents modify production without guardrails. The Stanford research shows AI-generated code is measurably less secure while making developers more confident about it.

A rewrite is a point-in-time event. Software is a continuous process. Agent-built rewrites solve the point-in-time problem. The continuous process — maintenance, evolution, operational resilience — remains open.

We spent 50 years treating code as the durable artifact and tests as disposable scaffolding. That relationship is inverting. Specifications and tests are becoming the permanent layer. Code is becoming the generated, replaceable output. The question worth asking isn't whether to rewrite. It's whether the specification you've accumulated — in tests, in docs, in domain experts' heads — is good enough to survive the rewrite. And if not: what would it take to make it so?

---

*Sources: [urlx](https://github.com/jonwiggins/urlx) and [xmloxide](https://github.com/jonwiggins/xmloxide) by Jon Wiggins ([HN](https://news.ycombinator.com/item?id=47490735), [HN](https://news.ycombinator.com/item?id=47201816)); [Anthropic C compiler](https://www.anthropic.com/engineering/building-c-compiler) by Nicholas Carlini ([GitHub](https://github.com/anthropics/claudes-c-compiler)); [Cursor scaling agents](https://cursor.com/blog/scaling-agents); [GitHub spec-kit](https://github.com/github/spec-kit); [Simon Willison](https://simonwillison.net/2026/Mar/14/pragmatic-summit/) on TDD with agents and [StrongDM Software Factory](https://simonwillison.net/2026/Feb/7/software-factory/); [Martin Fowler on SDD](https://martinfowler.com/articles/exploring-gen-ai/sdd-3-tools.html); [Joel Spolsky](https://www.joelonsoftware.com/2000/04/06/things-you-should-never-do-part-i/); [Andrej Karpathy on vibe coding](https://twitter.com/karpathy/status/1886192184808149383); [Perry et al., ACM CCS 2023](https://arxiv.org/abs/2211.03622) on AI code security; Daniel Stenberg on [dropping hyper](https://daniel.haxx.se/blog/2024/12/21/dropping-hyper/); Alibaba/Sun Yat-sen maintenance study via [Gary Marcus](https://garymarcus.substack.com/); [Hillel Wayne on LLMs and specifications](https://buttondown.com/hillelwayne/archive/llms-are-bad-at-vibing-specifications/); [AWS AI coding incident](https://the-decoder.com/aws-ai-coding-tool-decided-to-delete-and-recreate-a-customer-facing-system-causing-13-hour-outage-report-says/).*
