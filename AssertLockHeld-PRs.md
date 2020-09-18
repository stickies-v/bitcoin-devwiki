Wiki page to compare different PRs changing [`AssertLockHeld`](https://github.com/bitcoin/bitcoin/blob/be3af4f31089726267ce2dbdd6c9c153bb5aeae1/src/sync.h#L79). Please edit this page!

### Summary of approaches

- **1A** *One assert* [#19865](https://github.com/bitcoin/bitcoin/pull/19865): Removes runtime asserts and uses only compile time warnings, only keeping runtime asserts in cases where compile time checks don't work. Gets rid of multiple assert implementations. `AssertLockHeld` is the only one and it is restored to have the same definition it had from [2018](https://github.com/bitcoin/bitcoin/pull/13423) until [recently](https://github.com/bitcoin/bitcoin/pull/19668).

- **2A** *Two asserts* [#19918](https://github.com/bitcoin/bitcoin/pull/19918): Keeps runtime checks and uses two assert implementations instead of one: `AssertLockHeld` and `WeaklyAssertLockHeld`. The names are intentionally chosen so people favor the strong assertion instead of the weak assertion whenever possible, and there's never a question about which is better to use.

- **PA** *Proper asserts* [#19929](https://github.com/bitcoin/bitcoin/pull/19929): An approach that applied thread safety annotations as documented to avoid cases where the compiler might make incorrect assumptions.

- **QF** *Quick fix* [#19970](https://github.com/bitcoin/bitcoin/pull/19970) just fixes the bugs in LockAssertion (so no need for unused variable names, and file/line numbers are reported correctly).

### Underlying issues

I (@ajtowns) think there are four underlying issues here:

1. LockAssertion has some non thread-safety bugs, namely it reports the wrong file/line numbers on errors, and requires naming a dummy variable.
2. LockAssertion uses a dummy SCOPED_LOCKABLE rather than ASSERT_EXCLUSIVE_LOCK. 
3. Whether we should have redundant runtime checks in functions that already have compile time checks
4. What "LockAssertion" should be called

### Comparison of approaches

#### Advantages of current approach

- LockAssertion gives a compile error if it's used unnecessarily (`init.cpp:1548:13: error: acquiring mutex 'cs_main' that is already held [-Werror,-Wthread-safety-analysis] ... init.cpp:1547:64: note: mutex acquired here`)
- LockAssertion doesn't hide errors in some cases that we probably don't care about (assertion is inside a try block, an exception prior to the assertion is caught via a catch clause and the lock might not be held via that path, and something needing the lock is accessed after the try/catch block)
- AssertLockHeld() and AssertLockNotHeld() behave in the same way, in that they're enforced both by clang at compile time and checked at runtime when DEBUG_LOCKORDER is enabled

#### Disadvantages of current approach

- It has some non-thread-safety related bugs
- An upstream clang dev has [frowned on the dummy SCOPED_LOCKABLE approach](https://reviews.llvm.org/D87629#2272676) ("please don't use ACQUIRE when the capability is assumed to be held previously.")

#### Advantages of QF Approach

- Addresses (1)
- Fixes easy bugs quickly
- Doesn't make anything worse

#### Disadvantages of QF Approach

- Doesn't fix everything.
- Additional changes needed / rebasing needed on other PRs.

#### Advantages of 1A Approach

- Addresses (1), (2), (3), renames `LockAssertion` to `LockAlreadyHeld`
- Gets rid of `AssertLockHeld` calls which the compiler guarantees can never trigger at runtime, and which are not applied consistently in existing code
- Gets rid of `LockAssertion` class which is easily confused with `AssertLockHeld`, declares unused variable names, reports line numbers incorrectly, and is broken according to clang developers
- Falls back to runtime checks infrequently only where compile time checks don't work, and only requires a single assert macro `AssertLockHeld` 

#### Disadvantages of 1A Approach

- Will only detect problems if using Clang and configured with `--enable-debug` (not enabled by default).
- Problems are reported in the form of compile time warnings which can be missed unless configured with `--enable-werror` (not enabled by default).
- May not detect problems if [Clang has bugs](https://github.com/bitcoin/bitcoin/pull/19865#issuecomment-687604066). There is already some strange behavior that the amount of warnings produced [depends on the order of the attributes](https://github.com/bitcoin/bitcoin/pull/19668#discussion_r467244459).
- [Known limitations exist](https://clang.llvm.org/docs/ThreadSafetyAnalysis.html#limitations).
- Intrusive patch that touches lots of code

#### Advantages of 2A Approach

- Addresses (1), (2), renames `LockAssertion` to `WeaklyAssertLockHeld`
- Gets of broken `LockAssertion` class similar to 1A above
- Requires two different assert implementations `AssertLockHeld` and `WeaklyAssertLockHeld` but uses naming to indicate stronger assert should be preferred and adds documentation to help explain what they each do.
- Unlike 1A approach, does not drop runtime checks. This means if compile time checking is broken or disabled and thread sanitizer is broken or disabled, there is an extra level of checking
- Redundant `AssertLockHeld` calls may help with readability because unlike `EXCLUSIVE_LOCKS_REQUIRED` annotations you can see them in the body of the function, not just attached to the function declaration.

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
  | QF approach                                                  | LOCK_ASSERTION                  | AssertLockHeld                     |
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
