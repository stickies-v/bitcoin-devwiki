Wiki page to compare different PRs changing [`AssertLockHeld`](https://github.com/bitcoin/bitcoin/blob/be3af4f31089726267ce2dbdd6c9c153bb5aeae1/src/sync.h#L79). Please edit this page!

### Summary of approaches

- **1A** *One assert* [#19865](https://github.com/bitcoin/bitcoin/pull/19865): Gets rid of runtime asserts that are redundant with compile time checks, only keeping them in cases where compile time checks don't work. Gets rid of multiple assert implementations. `AssertLockHeld` is the only one and it is restored to have the same definition it had from [2018](https://github.com/bitcoin/bitcoin/pull/13423) until [recently](https://github.com/bitcoin/bitcoin/pull/19668).

- **2A** *Two asserts* [#19918](https://github.com/bitcoin/bitcoin/pull/19918): Keeps runtime checks and uses two assert implementations instead of one: `AssertLockHeld` and `WeaklyAssertLockHeld`. The names are intentionally chosen so people favor the strong assertion instead of the weak assertion whenever possible, and there's never a question about which is better to use.

- **PA** *Proper asserts* [#19929](https://github.com/bitcoin/bitcoin/pull/19929): PR was closed, but this was a more proper approach that applied thread safety annotations conservatively to avoid cases where the compiler might make incorrect assumptions.

- **AJA** *AJ asserts* [[1]](https://github.com/bitcoin/bitcoin/pull/19918#discussion_r485102739)[[2]](https://github.com/bitcoin/bitcoin/pull/19918#discussion_r488282255)[[3]](https://github.com/bitcoin/bitcoin/pull/19918#discussion_r490472714): Same as 2A, except calls it  `LOCK_ALREADY_HELD` instead of `WeaklyAssertLockHeld`.

### Comparison of approaches

#### Advantages of 1A Approach

- Gets rid of `AssertLockHeld` calls which the compiler guarantees can never trigger at runtime, and which are not applied consistently in existing code
- Gets rid of `LockAssertion` class which is easily confused with `AssertLockHeld`, declares unused variable names, reports line numbers incorrectly, and is [broken according to clang developers](https://reviews.llvm.org/D87629#2272676) ("please don't use ACQUIRE when the capability is assumed to be held previously.")
- Falls back to runtime checks infrequently only where compile time checks don't work, and only requires a single assert macro `AssertLockHeld` 

#### Disadvantages of 1A Approach

- May not detect race conditions or deadlocks if compile time checks are broken or disabled and thread sanitizer is broken or disabled. Compile time checks always run on CI but are not supported by all compilers locally

#### Advantages of 2A Approach

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
  | Pre-[#19668](https://github.com/bitcoin/bitcoin/pull/19668)  | AssertLockHeld & LockAnnotation | _(doesn't exist)_                  |
  | Post-[#16668](https://github.com/bitcoin/bitcoin/pull/19668) | LockAnnotation                  | AssertLockHeld                     |
  | 1A approach                                                  | AssertLockHeld                  | _(doesn't exist)_                  |
  | 2A approach                                                  | WeaklyAssertLockHeld            | AssertLockHeld                     |
  | AJA approach                                                 | LOCK_ALREADY_HELD               | AssertLockHeld                     |     
  | Alternate suggestion                                         | RuntimeAssertLockHeld           | AssertLockHeld                     |
  | Alternate suggestion                                         | RuntimeAssertLockHeld           | CompileTimeAssertLockHeld          |
  | Alternate suggestion                                         | AssertLockHeld                  | RedundantlyAssertLockHeld          |
  | Alternate suggestion                                         | UnprovenAssertLockHeld          | ProvenAssertLockHeld               |
  | Other suggestions?                                           |                                 |                                    |

#### Advantages of PA Approach

- More conservative and potentially avoids problems with false assumptions made by new compilers or future compiler versions

#### Disadvantages of PA Approach

- More complexity, various practical drawbacks [#19929 (comment)](https://github.com/bitcoin/bitcoin/pull/19929#issuecomment-690358411)

#### Advantages of AJA Approach

- If you need to use a lock, you have three options:
    * annotate the function with `EXCLUSIVE_LOCKS_REQUIRED(cs)` (and call `AssertLockHeld(cs)`)
    * write `LOCK(cs)` to acquire the lock
    * write `LOCK_ALREADY_HELD(cs)` if the lock is provably already held any time the function is called, but the function can't be annotated with EXCLUSIVE_LOCKS_REQUIRED
  In my opinion, calling `LOCK_ALREADY_HELD(cs)` guards the code following it in much the same was as `LOCK(cs)` does -- it's just that it's a guard that's validated manually prior to compile time, rather than at runtime by waiting on a mutex.
- Capitalising it draws attention, which is appropriate given the logic that the lock is always held needs to be validated by hand.

#### Disadvantages of AJA Approach
- Assert name doesn't contain the word assert. (Could call it LOCK_ASSERTION(cs) I guess)
- Both assert implementations have the exact same behavior at runtime, but different names. It's may be confusing which assert is better to use if both are accepted by the compiler.  (Using a dummy SCOPED_LOCKABLE instead of ASSERT_EXCLUSIVE_LOCK ensures that AssertLockHeld and the current LockAssertion can only be used when appropriate; you'll get a compile error if AssertLockHeld is invoked when the lock isn't held, and likewise if you create a LockAssertion when the lock is held, or if you LOCK the mutex later in the same scope as the LockAssertion)