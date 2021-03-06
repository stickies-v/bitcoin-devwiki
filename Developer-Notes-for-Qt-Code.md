### Contents

1. [Git Tips](#git-tips)
1. [Backward Compatibility](#backward-compatibility)
1. [`QObject` Subclassing Style](#qobject-subclassing-style)
1. [Debugging Tips](#debugging-tips)

## Git Tips

The [GUI repo](https://github.com/bitcoin-core/gui) and the main [Bitcoin Core repo](https://github.com/bitcoin/bitcoin)
share the same commit history. If you are going to contribute, including reviewing, to both of the repo, it is
recommended to have them locally.

### GUI Repo Setup

1. Log in [GitHub](https://github.com) (in command examples the GitHub username _"satoshi"_ used), open the [GUI repo](https://github.com/bitcoin-core/gui), and [fork](https://docs.github.com/en/github/getting-started-with-github/fork-a-repo) it.

2. Clone the repo locally:
```sh
$ git clone https://github.com/bitcoin-core/gui.git && cd gui
```

3. Set up your forked repo as a local git remote:
```sh
$ git remote add satoshi https://github.com/satoshi/gui.git
$ git config remote.pushDefault satoshi
```

The current result could be verified:
```sh
$ git remote -v
origin	https://github.com/bitcoin-core/gui.git (fetch)
origin	https://github.com/bitcoin-core/gui.git (push)
satoshi	https://github.com/satoshi/gui.git (fetch)
satoshi	https://github.com/satoshi/gui.git (push)
```

4. To synchronize (any time you want) your local and forked repos:
```sh
$ git switch master
$ git pull --ff-only
$ git push
```

Note: Just after cloning repos are already synchronized.

### Pull Request Reviewing

The recommended way to review a pull request (PR) is do it locally because it allows you to build binaries and test them.
In command examples [PR 42](https://github.com/bitcoin-core/gui/pull/42) is used.

Fetch PR branch to the local repo:
```sh
$ git fetch origin pull/42/head:pr42.01 && git switch pr42.01
```

It is useful to use separated local branches for every fetching. Optionally, branch names could include a sequence number
(as in example above) or a convenient date/time presentation.

When a PR author updates her branch, it is easy to check the changes:
```sh
$ git diff pr42.03 pr42.04
```

Even if a PR branch was rebased, it is still easy to check the changes:
```sh
$ git range-diff master pr42.03 pr42.04
```

Note: Git [branches are cheap to create and destroy](https://git-scm.com/book/en/v2/Git-Branching-Branches-in-a-Nutshell#ch03-git-branching).

After PR merging, all of the review branches could be removed:
```sh
$ git switch master
$ git branch -D $(git branch | grep pr42)
```

## Backward Compatibility

The source code must be compatible with the [minimum required](https://github.com/bitcoin/bitcoin/blob/master/doc/dependencies.md) Qt version which is set in `configure.ac`:
```sh
BITCOIN_QT_CONFIGURE([5.9.5])
```

If an optional feature requires a higher version of Qt, or if a feature was replaced by another one, use the `QT_VERSION` and `QT_VERSION_CHECK` macros:

```cpp
#include <QDate>
#include <QDateTime>
#include <QtGlobal>  // For QT_VERSION and QT_VERSION_CHECK macros.

QDateTime StartOfDay(const QDate& date)
{
#if (QT_VERSION >= QT_VERSION_CHECK(5, 14, 0))
    return date.startOfDay();  // QDate::startOfDay was introduced in Qt 5.14.0.
#else
    return {date};
#endif
}

```

Do not compare versions of `QT_VERSION` directly using `0xMMNNPP` (MM = major, NN = minor, PP = patch), as this approach is less readable and more error-prone.

Every time the minimum required Qt version is bumped, `grep` or `git grep` for all of the `QT_VERSION` instances and adjust/remove them accordingly.

## `QObject` Subclassing Style

When sublassing the `QObject` class follow the pattern below:

```cpp
#include <QObject>

class MyWidget : public QObject
{
    Q_OBJECT

public:
    // Public members.

public Q_SLOTS:
    // Public slots.

Q_SIGNALS:
    // Signals (inherently public).

private:
    // Private members.

private Q_SLOTS:
    // Private slots.
};
```

Use the `Q_SIGNALS` and `Q_SLOTS` macros instead of the `signals` and `slots` keywords of the Qt `moc` (Meta-Object Compiler). This prevents potential conflicts with 3rd party signal/slot mechanisms.

## Debugging Tips

For debugging, including signal-to-slot connection issues, you can use the `QT_FATAL_WARNINGS` environment variable:

```sh
$ QT_FATAL_WARNINGS=1 src/qt/bitcoin-qt -printtoconsole -debug=qt
```

This tip can be [combined](https://github.com/bitcoin/bitcoin/pull/16118#issuecomment-503184695) with a debugger.

***

_More notes come soon..._