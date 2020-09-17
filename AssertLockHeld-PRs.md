Wiki page to compare different PRs changing [`AssertLockHeld`](https://github.com/bitcoin/bitcoin/blob/be3af4f31089726267ce2dbdd6c9c153bb5aeae1/src/sync.h#L79). Please edit this page!

### Summary of approaches

- **1A** [#19865](https://github.com/bitcoin/bitcoin/pull/19865): Gets rid of runtime asserts that are redundant with compile time checks, only keeping them in cases where compile time checks don't work. Gets rid of multiple assert implementations. `AssertLockHeld` is the only one and it is restored to have the same definition it had from [2018](https://github.com/bitcoin/bitcoin/pull/13423) until [recently](https://github.com/bitcoin/bitcoin/pull/19668).

- **2A** [#19918](https://github.com/bitcoin/bitcoin/pull/19918): Keeps runtime checks and uses two assert implementations instead of one: `AssertLockHeld` and `WeaklyAssertLockHeld`. The names are intentionally chosen so people favor the strong assertion instead of the weak assertion whenever possible, and there's never a question about which is better to use.

- **PA** [#19929](https://github.com/bitcoin/bitcoin/pull/19929): PR was closed, but this was a more proper approach that applied thread safety annotations conservatively to avoid cases where the compiler might make incorrect assumptions.

- **AJA** [[1]](https://github.com/bitcoin/bitcoin/pull/19918#discussion_r485102739)[[2]](https://github.com/bitcoin/bitcoin/pull/19918#discussion_r488282255)[[3]](https://github.com/bitcoin/bitcoin/pull/19918#discussion_r490472714): Adds `LOCK_ALREADY_HELD` macro, _unclear what summary of this approach is_

### Comparison of approaches

#### Advantages of 1A Approach

- Gets rid of `AssertLockHeld` calls which the compiler guarantees can never trigger at runtime, and which are not applied consistently in existing code
- Gets rid of `LockAssertion` class which is easily confused with `AssertLockHeld`, declares unused variable names, reports line numbers incorrectly, and is [broken according to clang developers](https://reviews.llvm.org/D87629#2272676) ("please don't use ACQUIRE when the capability is assumed to be held previously.")
- Use runtime checks infrequently only where compile time checks don't work, and only requires a single assert macro `AssertLockHeld` 
, there maybe race conditions or deadlocks
#### Disadvantages of 1A Approach

- Not supported by all compilers
- May not detect race conditions or deadlocks if compile time checks are broken or disabled and thread sanitizer is broken or disabled

#### Advantages of 2A Approach

- Gets of broken `LockAssertion` class similar to 1A above
- Requires two different assert implementations `AssertLockHeld` and `WeaklyAssertLockHeld` but uses naming to indicate stronger assert should be preferred and adds documentation to help explain what they each do.
- Unlike 1A approach, does not drop runtime checks, and keeps developer guide encouragement make runtime checking more consistent in the future. This means if compile time checking is broken or disabled and thread sanitizer is broken or disabled, there is an extra level of checking
- Redundant `AssertLockHeld` calls may help with readability because unlike `EXCLUSIVE_LOCKS_REQUIRED` annotations you can see them in the body of the function not just attached to the function declaration.

#### Disadvantages of 2A Approach

- Requires two different assert implementations instead of one and more complicated developer guidelines.

#### Advantages of PA Approach

- More conservative and potentially avoids problems with false assumptions made by new compilers or future compiler versions.

#### Disadvantages of PA Approach

- More complexity, various practical drawbacks [#19929 (comment)](https://github.com/bitcoin/bitcoin/pull/19929#issuecomment-690358411)

#### Advantages of AJA Approach

#### Disadvantages of AJA Approach
- Assert name doesn't contain the word assert.
- Two assert calls have same behavior at runtime, and it may be unclear which call to prefer