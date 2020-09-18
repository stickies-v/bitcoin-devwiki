Wiki page to compare different PRs changing [`AssertLockHeld`](https://github.com/bitcoin/bitcoin/blob/be3af4f31089726267ce2dbdd6c9c153bb5aeae1/src/sync.h#L79). Please edit this page!

### Summary of approaches

- **1A** *One assert* [#19865](https://github.com/bitcoin/bitcoin/pull/19865): Removes runtime asserts and uses only compile time warnings, only keeping runtime asserts in cases where compile time checks don't work. Gets rid of multiple assert implementations. `AssertLockHeld` is the only one and it is restored to have the same definition it had from [2018](https://github.com/bitcoin/bitcoin/pull/13423) until [recently](https://github.com/bitcoin/bitcoin/pull/19668).

- **2A** *Two asserts* [#19918](https://github.com/bitcoin/bitcoin/pull/19918): Keeps runtime checks and uses two assert implementations instead of one: `AssertLockHeld` and `WeaklyAssertLockHeld`. The names are intentionally chosen so people favor the strong assertion instead of the weak assertion whenever possible, and there's never a question about which is better to use.

- **PA** *Proper asserts* [#19929](https://github.com/bitcoin/bitcoin/pull/19929): Currently abandoned/closed approach that applied thread safety annotations as documented to avoid cases where the compiler might make incorrect assumptions.

- **QFA** *Quick fix asserts* [#19970](https://github.com/bitcoin/bitcoin/pull/19970): fixes usability issues in LockAssertion (so no need for unused variable names, and file/line numbers are reported correctly) and introduces a new LOCK_ASSERTION macro to wrap it

- **NA** *Naked asserts* [[1]](http://www.erisian.com.au/bitcoin-core-dev/log-2020-09-17.html#l-650)[[2]](https://reviews.llvm.org/D87629#2272676)[[3]](https://reviews.llvm.org/D87629#2278073): Don't annotate AssertLockHeld() with any compile time attributes and leave it pure run time. This change is orthogonal and can be combined with any of the approaches above

### Underlying issues

- Currently we have both [`AssertLockHeld`](https://github.com/bitcoin/bitcoin/blob/be3af4f31089726267ce2dbdd6c9c153bb5aeae1/src/sync.h#L79) and [`LockAssertion`](https://github.com/bitcoin/bitcoin/blob/be3af4f31089726267ce2dbdd6c9c153bb5aeae1/src/sync.h#L357) and it is confusing what the differences are between them and when each should be used.

- AssertLockHeld is currently used haphazardly in the code. The developer notes [recommend](https://github.com/bitcoin/bitcoin/blob/master/doc/developer-notes.md#threads-and-synchronization) that it is used every place that EXCLUSIVE_LOCK_FUNCTION is used, but the recommendation is not enforced or consistently followed. (There is also disagreement about whether the recommendation is useful on its merits. The asserts may help with readability and may help detect bugs during development when compiling locally, but they add verbosity to the code and catch fewer errors than the static checks enforced by the project's QA)

- LockAssertion class is using the [EXCLUSIVE_LOCK_FUNCTION](https://clang.llvm.org/docs/ThreadSafetyAnalysis.html#acquire-acquire-shared-release-release-shared-release-generic) acquire annotation incorrectly: ["please don't use ACQUIRE when the capability is assumed to be held previously"](https://reviews.llvm.org/D87629#2272676)

- LockAssertion class has minor usability issues. It reports the wrong file/line numbers on errors, and requires naming a dummy variable.

### Comparison of approaches

#### Advantages of 1A Approach

- One type of assert and not two. No confusion!
- Gets rid of `AssertLockHeld` calls which the compiler guarantees can never trigger at runtime, and which are not applied consistently in existing code
- Gets rid of `LockAssertion` class which is easily confused with `AssertLockHeld`, declares unused variable names, reports line numbers incorrectly, and is broken according to clang developers
- Falls back to runtime checks infrequently only where compile time checks don't work, and only requires a single assert macro `AssertLockHeld` 
- Fixes LockAssertion annotation bug and usability problems by deleting LockAssertion

#### Disadvantages of 1A Approach

- Will only detect problems locally if using Clang and configured with `--enable-debug` (not enabled by default). Checks are enforced on every PR and on the master branch in QA.
- Problems are reported in the form of compile time warnings which can be missed unless configured with `--enable-werror` (not enabled by default).
- May not detect problems if [Clang has bugs](https://github.com/bitcoin/bitcoin/pull/19865#issuecomment-687604066). There is already some strange behavior that the amount of warnings produced [depends on the order of the attributes](https://github.com/bitcoin/bitcoin/pull/19668#discussion_r467244459).
- [Known limitations exist](https://clang.llvm.org/docs/ThreadSafetyAnalysis.html#limitations).
- Despite being a 3-line scripted diff, it is an intrusive patch that touches lots of code

#### Advantages of 2A Approach

- Gets of broken `LockAssertion` class similar to 1A above
- Requires two different assert implementations `AssertLockHeld` and `WeaklyAssertLockHeld` but uses naming to indicate stronger assert should be preferred and adds documentation to help explain what they each do.
- Unlike 1A approach, does not drop runtime checks. This means if compile time checking is broken or disabled and thread sanitizer is broken or disabled, there is an extra level of checking
- Redundant `AssertLockHeld` calls may help with readability because unlike `EXCLUSIVE_LOCKS_REQUIRED` annotations you can see them in the body of the function, not just attached to the function declaration.
- Fixes LockAssertion annotation bug and usability problems by deleting LockAssertion

#### Disadvantages of 2A Approach

- Requires two different assert implementations instead of one.
- Unlike 1A approach, does not clean up inconsistent runtime assertions in current code. It keeps developer notes recommendation to add them more places.
- `WeaklyAssertLockHeld` name may be [confusing](https://github.com/bitcoin/bitcoin/pull/19918#issuecomment-694486228). Since both asserts do exactly the same thing at runtime and only compile time annotations differ, different naming schemes are possible. Feel free to add suggestions below:

  |                                                              | ASSERT_EXCLUSIVE_LOCK assertion | EXCLUSIVE_LOCKS_REQUIRED assertion |
  |--------------------------------------------------------------|---------------------------------|------------------------------------|
  | Pre-[#13423](https://github.com/bitcoin/bitcoin/pull/13423)  | (doesn't exist)                 | AssertLockHeld (if lock is used)   |
  | Post-[#13423](https://github.com/bitcoin/bitcoin/pull/13423) | AssertLockHeld                  | _(doesn't exist)_                  |
  | Post-[#14437](https://github.com/bitcoin/bitcoin/pull/14437) | AssertLockHeld & LockAnnotation | _(doesn't exist)_                  |
  | Post-[#16034](https://github.com/bitcoin/bitcoin/pull/16034) | AssertLockHeld & LockAssertion  | _(doesn't exist)_                  |
  | Post-[#19668](https://github.com/bitcoin/bitcoin/pull/19668) | LockAssertion                   | AssertLockHeld                     |
  | 1A approach                                                  | AssertLockHeld                  | _(doesn't exist)_                  |
  | 2A approach                                                  | WeaklyAssertLockHeld            | AssertLockHeld                     |
  | QFA approach                                                 | LOCK_ASSERTION                  | AssertLockHeld                     |
  | Alternate suggestion                                         | LOCK_ALREADY_HELD               | AssertLockHeld                     |     
  | Alternate suggestion                                         | RuntimeAssertLockHeld           | AssertLockHeld                     |
  | Alternate suggestion                                         | RuntimeAssertLockHeld           | CompileTimeAssertLockHeld          |
  | Alternate suggestion                                         | AssertLockHeld                  | RedundantlyAssertLockHeld          |
  | Alternate suggestion                                         | UnprovenAssertLockHeld          | ProvenAssertLockHeld               |
  | Alternate suggestion                                         | UnsafelyAssertLockHeld          | AssertLockHeld                     |
  | Alternate suggestion                                         | UnprovablyAssertLockHeld        | AssertLockHeld                     |
  | Other suggestions?                                           |                                 |                                    |


#### Advantages of PA Approach

- Uses thread safety annotations as documented and potentially avoids problems with false assumptions made by new compilers or future compiler versions

#### Disadvantages of PA Approach

- Various practical drawbacks [#19929 (comment)](https://github.com/bitcoin/bitcoin/pull/19929#issuecomment-690358411)

#### Advantages of QFA Approach

- Addresses LockAssertion usability issues
- Fixes easy bugs quickly
- Doesn't make anything worse

#### Disadvantages of QFA Approach

- Doesn't address LockAssertion annotation misuse
- Additional changes needed / rebasing needed on other PRs.
- Introduces a new macro

#### Advantages of NA Approach

- Conceptually simpler to keep runtime & compile time checks separate and allow AssertLockHeld to be use freely in any context without having any impact on the compiler

#### Disadvantages of NA Approach

- Hurts readability. If AssertLockHeld is annotated with ASSERT_EXCLUSIVE_LOCK a reader can be sure that the compiler has verified the lock is held at the point of the assert. If it's not annotated, the assertion is an unproven claim that has only been verified by runtime tests passing

- Less safe. AssertLockHeld has a greater likelihood of aborting if the compiler hasn't proven the lock is held at the location