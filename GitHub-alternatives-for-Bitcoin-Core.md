# Motivation

## Basic Usability Issues

### Overall Unreliability

GitHub makes comments in long threads inaccessible via it's site. https://twitter.com/jfnewbery/status/1354744697093283847

The comments need to be recovered manually from past emails or metadata dumps.

It was also observed that complete issues or pull requests simply disappeared without notice.

### Spam/Lack of Moderator Queue

The repo has thousands of subscribers, so any spam contribution or otherwise inappropriate comment will reach all those inboxes before being deleted by a maintainer.

GitHub fails to provide simple and effective tools (such as rate limits for non-members or a moderator queue) to fight low-quality content.

https://twitter.com/MarcoFalke/status/1355097159113322497

### Others

```
https://github.com/bitcoin/bitcoin/issues/15847 | Feedback for GitHub CEO · Issue #15847 · bitcoin/bitcoin · GitHub
https://github.com/bitcoin/bitcoin/issues/20227 | Dependency on GitHub · Issue #20227 · bitcoin/bitcoin · GitHub
 · Issue #16472 · bitcoin/bitcoin · GitHub
```

## Centralization/Single Point of Failure

The repo is primarily accessed through GitHub, which might (even against their will) shut down hosting at any point in time.

Instead of hopping to another centralized provider or self-hosted single point of failure, a goal should be to evaluate decentralized or federated hosters. See also:

* https://github.com/bitcoin/bitcoin/issues/13411 (Moving to self-hosted issue and patch management)
* https://github.com/bitcoin/bitcoin/issues/16472 (Github started banning/restricting whole countries)

# Alternatives

Possible alternatives:

- [git-appraise](https://github.com/google/git-appraise)
- [Radicle](https://radicle.xyz/)
- [git-bug](https://github.com/MichaelMure/git-bug)
- [bugs-everywhere](https://bugs-everywhere.readthedocs.io/en/latest/tutorial.html)
- [Darcs](http://darcs.net)