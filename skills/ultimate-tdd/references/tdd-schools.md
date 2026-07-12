# TDD Schools — London, Chicago, and Buenos Aires

RED–GREEN–REFACTOR explains how to advance one increment at a time, but it
does not determine **where development should begin**. Three schools of TDD
address this: where you start, which direction you grow, and how you isolate
collaborators along the way.

## The Three Schools

| School | Direction | Isolation style | Origin |
|--------|-----------|-----------------|--------|
| **Chicago** (Detroit / Classicist) | Inside-Out | State verification; real objects by default | Kent Beck, Chrysler C3 project |
| **London** (Mockist) | Outside-In | Behavior verification; doubles for every collaborator with interesting behavior | Steve Freeman, Nat Pryce, London Extreme Tuesday Club |
| **Buenos Aires** (Middle-Out) | Middle-Out | Hybrid — doubles at awkward edges, real objects for logic; direction follows the next meaningful failure | Hernán Wilkinson |

## Direction and isolation are independent decisions

- **Direction** — where to start and which way to grow (outside-in, inside-out, middle-out).
- **Isolation style** — whether to use real collaborators or doubles, and whether to verify by state or behavior.

Outside-in does **not** require mocking every layer. Inside-out does **not** prohibit test doubles. These choices are related but independent.

## Default Decision Policy

**Use outside-in** when implementing new behavior primarily defined from the perspective of a user, API consumer, message consumer, CLI caller, or another external actor.

**Use inside-out** when the main uncertainty lies in domain rules, calculations, state transitions, parsers, algorithms, transformations, or invariants that can be expressed independently from delivery and infrastructure concerns.

**Use middle-out (Buenos Aires)** for most production features — a hybrid double loop that combines the strengths of both. See Middle-Out Loop below.

## Decision Table

| Situation | Preferred starting point | Why | Guardrail |
|-----------|-------------------------|-----|-----------|
| New user-facing feature spanning several layers | Outside-in | Keeps implementation connected to an observable requirement and exposes integration assumptions early | Begin with one thin happy path, not an exhaustive end-to-end suite |
| New API endpoint, command, event consumer, or workflow | Outside-in | Lets the external contract drive the internal design | Test through the narrowest stable public boundary |
| Rich domain rules, calculations, policies, or state machines | Inside-out | Domain examples usually provide the fastest and clearest feedback | Add an outer test before declaring the feature complete |
| Algorithm, parser, formatter, validator, or pure transformation | Inside-out | External layers add little information while increasing test cost | Test public behavior rather than private helper methods |
| New service or architecture with uncertain wiring | Walking skeleton followed by outside-in | Validates build, runtime, deployment, communication paths, and major boundaries early | Implement the thinnest executable path; do not build speculative abstractions |
| Database, queue, serializer, filesystem, or third-party protocol behavior | Boundary integration or contract test | A mock cannot prove that schemas, queries, serialization, transactions, or protocols are correct | Use real infrastructure locally when practical, otherwise use a faithful fake or contract test |
| Bug in an existing system | Start at the lowest stable boundary that reproduces the bug | Produces a focused and durable regression test | Add a broader test only when the defect was caused by integration or wiring |
| Legacy code with no useful seams | Characterization test around current observable behavior | Establishes safety before restructuring | Do not redesign and add the feature simultaneously |
| Small CRUD behavior mostly provided by a framework | Thin outside-in or component test plus focused integration tests | Per-class mocks often duplicate framework wiring without increasing confidence | Do not create controller, service, and repository unit tests automatically |
| Reusable low-level library with no established application workflow | Inside-out | Its contract is the library API rather than an application acceptance flow | Drive from public API examples, not implementation details |

## Outside-In Loop (London)

1. Write the smallest acceptance or component test that describes one externally observable behavior.
2. Run it and verify that it fails for the expected reason.
3. Follow the failure inward.
4. When a missing lower-level behavior becomes clear, write a focused inner test for it.
5. Complete a RED–GREEN–REFACTOR cycle at that level.
6. Return to the outer test.
7. Repeat until the outer test passes.

Do not design all layers in advance. The next meaningful failure should determine the next implementation step.

An outer acceptance test may remain red across several inner RED–GREEN–REFACTOR cycles. "One failing test at a time" applies to the active inner design step; the outer test is intentionally retained as the feature-progress test.

## Inside-Out Loop (Chicago)

1. Select the smallest domain behavior or public component contract required by the feature.
2. Express it with an example using domain language.
3. Complete RED–GREEN–REFACTOR using real, lightweight collaborators where practical.
4. Grow outward only when an external adapter or orchestration layer has a concrete need for the behavior.
5. Add integration and acceptance coverage before considering the vertical feature complete.

Do not build domain capabilities merely because they might be useful later. Every inner behavior must be justified by a known requirement, example, or external use case.

## Middle-Out Loop (Buenos Aires)

Middle-out starts at the **use case or application service layer** — the seam between external delivery and domain logic — and grows in both directions as tests demand.

The double loop:

1. Add one focused acceptance, component, or contract test describing the externally observable behavior.
2. Keep that outer test failing while using small RED–GREEN–REFACTOR cycles at lower levels.
3. Implement only the next collaboration or component required by the current failure.
4. Make the outer test pass.
5. Refactor the complete slice while all tests remain green.

The outer test measures feature progress. Inner tests provide fast and precise design feedback.

This is the default approach for most production features: it avoids the over-engineering risk of pure inside-out and the fragile-test risk of pure outside-in.

## Walking Skeleton

Before substantial feature development, use a walking skeleton when the architecture, deployment path, or integrations are uncertain.

A walking skeleton is the thinnest executable slice that:

- enters through a real system boundary;
- crosses the important architectural components;
- reaches any critical infrastructure boundary;
- produces one observable result;
- can be built and tested automatically;
- uses production-shaped wiring, while allowing controlled substitutes for unavailable external systems.

Its purpose is to validate the broad system shape and feedback pipeline, not to implement meaningful business completeness.

Start with one simple success case. Avoid adding comprehensive validation, error handling, abstractions, or multiple scenarios until the skeleton works.

## Test-Double Policy

Prefer a real collaborator when it is: fast, deterministic, local, easy to configure, safe to execute, and capable of producing clearer tests than a double.

Use a stub or fake when a collaborator is slow, nondeterministic, remote, unavailable, expensive, destructive, or difficult to place into the required state.

Use interaction-verifying mocks only when the interaction itself is part of the required behavior:
- sending a command or notification;
- publishing an event;
- committing or rolling back a unit of work;
- invoking a required protocol step;
- preventing a forbidden side effect.

Do not mock value objects, simple domain entities, collections, pure functions, or lightweight in-process collaborators merely to isolate one class.

Do not verify call order unless ordering is an actual protocol or business requirement.

Mock application-owned roles or ports rather than concrete internals of third-party libraries. Test third-party integrations through adapters with focused integration or contract tests.

For the full taxonomy of doubles (Dummy, Fake, Stub, Spy, Mock) and the classicist vs. mockist distinction, see [test-doubles.md](test-doubles.md).

## Required Boundary Coverage

Unit tests cannot demonstrate that the application correctly communicates with databases, filesystems, queues, HTTP services, serializers, or frameworks.

For each important boundary, add a focused integration or contract test covering the production-relevant behavior:

- serialization and deserialization;
- database queries and mappings;
- transaction semantics;
- queue or event schemas;
- HTTP request and response contracts;
- filesystem behavior;
- framework routing and dependency wiring.

Mocks may support fast inner-loop tests, but they must not be the only evidence that an external boundary works.

## Behavior Over Implementation

Regardless of direction:

- Test observable behavior, contracts, outputs, state transitions, or required side effects.
- Avoid tests that reproduce the internal class structure.
- Do not create one unit test suite per class automatically.
- Refactoring internal structure without changing behavior should not require widespread test rewrites.
- Prefer the lowest test level that provides the required confidence.
- Retain higher-level tests when they prove wiring, contracts, or behavior that lower-level tests cannot establish.

## Completion Rule

A vertical behavior is complete only when:

- its externally observable acceptance or component test passes;
- important domain behavior has focused fast tests;
- production-relevant boundaries have integration or contract coverage;
- unnecessary duplication and speculative code have been removed;
- the complete relevant test suite remains green.

## Sources

- Fowler, M. — "Mocks Aren't Stubs" (classicist vs mockist distinction)
- Freeman, S. & Pryce, N. — "Growing Object-Oriented Software, Guided by Tests" (London School / outside-in)
- Beck, K. — "Test-Driven Development: By Example" (Chicago School / inside-out)
- Uncle Bob Martin & Sandro Mancuso — "London vs. Chicago" (Clean Coders comparative case study)
