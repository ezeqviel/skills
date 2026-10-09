# Test doubles — Martin Fowler's taxonomy

Reference material for isolating the SUT from its collaborators during a TDD cycle.
Sources: [Fowler — *Mocks Aren't Stubs*](https://martinfowler.com/articles/mocksArentStubs.html)
and [*Test Double*](https://martinfowler.com/bliki/TestDouble.html) (original taxonomy from
Gerard Meszaros, *xUnit Test Patterns*).

> **Test Double** is the generic term for any pretend object that stands in for a real one
> during a test. The name comes from the *stunt double* in film.

The common confusion is calling all five a "mock". They aren't: in the taxonomy, a **mock**
is only one of the five types. Tooling feeds the confusion — vitest's `vi.fn()` is a "mock
function", Python's `unittest.mock` names everything "Mock" — but what you build with them is
usually a **stub** or a **spy**, not a mock.

---

## The five doubles

| Double | Definition (Fowler / Meszaros) | Verifies by | Typical use |
|---|---|---|---|
| **Dummy** | Passed around but never actually used; only fills a signature. | — | Filling parameters the path under test never touches. |
| **Fake** | A working implementation with a shortcut that makes it unfit for production. | state | In-memory DB, a test repository, `(key) => key` as a translator. |
| **Stub** | Provides canned answers; doesn't react to anything outside the script. | state | Forcing a collaborator's return value (`mockResolvedValue(...)`). |
| **Spy** | A stub that **also** records how it was called. | state (+ recording) | Counting invocations, capturing the payload it received. |
| **Mock** | Pre-programmed with **expectations**: the test fails if calls don't happen as specified. | behavior | Asserting that something was (not) called — when the call *is* the contract. |

Fowler, verbatim:

- **Dummy** — *"Dummy objects are passed around but never actually used. Usually they are just used to fill parameter lists."*
- **Fake** — *"Fake objects actually have working implementations, but usually take some shortcut which makes them not suitable for production (an in memory database is a good example)."*
- **Stub** — *"Stubs provide canned answers to calls made during the test, usually not responding at all to anything outside what's programmed in for the test."*
- **Spy** — *"Spies are stubs that also record some information based on how they were called."*
- **Mock** — *"Mocks are objects pre-programmed with expectations which form a specification of the calls they are expected to receive."*

---

## State verification vs. behavior verification

The distinction that really matters isn't between objects, but between **how you verify**.

- **By state** — you exercise the system under test (SUT) and examine the result or final state.
  *"We determine whether the exercised method worked correctly by examining the state of the SUT
  and its collaborators after the method was exercised."*
- **By behavior** — you assert that certain calls to collaborators occurred.

> *"Only mocks insist upon behavior verification. The other doubles can, and usually do, use
> state verification."*

Rule of thumb: verify by **state** by default (more robust against refactors); reserve
**behavior** verification for when the call itself is the contract — e.g. "don't call the
privileged client when the caller isn't authorized" is a security guarantee, not an
implementation detail.

---

## Classic vs. mockist TDD

- **Classic** — *"Use real objects if possible and a double if it's awkward to use the real
  thing."* You only double what's awkward (I/O, network, clock); everything else is real
  objects, verified by state.
- **Mockist** — *"A mockist TDD practitioner will always use a mock for any object with
  interesting behavior."* You double every collaborator with behavior, verifying by calls.

Fowler declares himself classic, and his reservation is the one that matters for long-term
maintenance:

> *"I've always been an old fashioned classic TDDer and thus far I don't see any reason to
> change. I don't see any compelling benefits for mockist TDD, and am concerned about the
> consequences of coupling tests to implementation."*

> *"Mockist tests are thus more coupled to the implementation of a method. Changing the nature
> of calls to collaborators usually cause a mockist test to break."*

And the hidden cost of doubling too much: *"Many people like the fact that client tests may
catch errors that the main tests for an object may have missed, particularly probing areas
where classes interact. Mockist tests lose that quality."*

The choice of direction (outside-in vs inside-out vs middle-out) affects which
isolation style you default to. See [tdd-schools.md](tdd-schools.md) for the
decision table and the three schools (London, Chicago, Buenos Aires).

---

## Operational guide

1. **The most useful double is the most faithful one, not the most flexible.** A double that
   answers anything (a `MagicMock` that auto-creates attributes) stops failing when it should —
   it masks the drift between the test and the code. Prefer an object that only has what you
   declared and fails loudly on what you didn't (`SimpleNamespace`, a dataclass, an explicit fake).
2. **Don't mock a collaborator's *shape*, mock its *output*.** Doubling a fluent builder
   (`from().select().eq().single()`) couples the test to how the collaborator talks to its
   dependency; the test breaks on a refactor that didn't change behavior. Stub the higher-level
   function by its result.
3. **Verify by state unless the call is the contract.** Keep mocks (behavior verification) for
   invariants where "was called / wasn't called" is the guarantee itself.
4. **Real objects when they're cheap.** A pure function, a validator, a mapper — no double
   needed. Double the edge (I/O), not the logic.

---

## In practice

Your tooling lies about names — the two rules below are the ones prose doesn't land.

**Double the output, not the shape.** Stub the collaborator by what it returns, not by
its call chain.

```js
// ✗ shape — couples the test to the query builder; a behavior-preserving
//   refactor of the query breaks it even though nothing observable changed
const db = { from: () => ({ select: () => ({ eq: () => ({ single: () => ({ data: user }) }) }) }) }

// ✓ output — stub the repository function by its result
const getUser = async (id) => user
```

**Classic over mockist.** Use the real object when it's cheap; assert on the result,
not on which calls happened.

```py
# ✗ mockist — asserts on the call, coupled to how the total is computed
pricing = Mock()
cart = Cart(pricing=pricing)
cart.add(item)
pricing.apply.assert_called_once_with(item)   # tests the wiring, not the outcome

# ✓ classic — real pricing (pure logic), assert on the resulting state
cart = Cart(pricing=Pricing())
cart.add(item)
assert cart.total() == 90
```

If the collaborator can't be swapped for a real object or a double, the SUT has no seam —
that's a design signal (inject the dependency), not a reason to reach for a heavier mock.

---

## Sources
- Fowler, M. — *Mocks Aren't Stubs*: https://martinfowler.com/articles/mocksArentStubs.html
- Fowler, M. — *Test Double* (bliki): https://martinfowler.com/bliki/TestDouble.html
- Meszaros, G. — *xUnit Test Patterns: Refactoring Test Code* (original taxonomy).
