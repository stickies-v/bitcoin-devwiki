### Contents

1. [Git Tips](#git-tips)
1. [Backward Compatibility](#backward-compatibility)
1. [`QObject` Subclassing Style](#qobject-subclassing-style)
1. [Writing Code for Translation](#writing-code-for-translation)
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

## Writing Code for Translation

It is assumed that developers follow those guides:
- [Translation Process](https://github.com/bitcoin/bitcoin/blob/master/doc/translation_process.md)
- [Translation Strings Policy](https://github.com/bitcoin/bitcoin/blob/master/doc/translation_strings_policy.md)

### Translation in C++ Code

1. To translate a string literal (quoted text) that will be presented to the user, use the [`QObject::tr`](https://doc.qt.io/qt-5.12/qobject.html#tr) function which returns a translated `QString` object:

- within classes that are derived from the `QObject`:
```cpp
QString need_sig_text = tr("Transaction still needs signature(s).");
```

- in other cases:
```cpp
QString NetworkToQString(Network net)
{
    switch (net) {
    case NET_UNROUTABLE: return QObject::tr("Unroutable");
    case NET_IPV4: return "IPv4";
    case NET_IPV6: return "IPv6";
    case NET_ONION: return "Onion";
    case NET_I2P: return "I2P";
    case NET_CJDNS: return "CJDNS";
    case NET_INTERNAL: return QObject::tr("Internal");
    case NET_MAX: assert(false);
    } // no default case, so the compiler can warn about missing cases
    assert(false);
}
```

2. Some substrings are unknown at the compile time or are not intended to be translated, e.g., error messages, command line options, default file names etc.
In such cases it is recommended:
- to avoid unwanted translation, and
- do not break the whole string into fragments those are translated separately

To achieve both goals, use numbered place markers with the [`QString::arg`](https://doc.qt.io/qt-5.12/qstring.html#arg) function:

```cpp
QString EditAddressDialog::getDuplicateAddressWarning() const
{
    QString dup_address = ui->addressEdit->text();
    QString existing_label = model->labelForAddress(dup_address);
    QString existing_purpose = model->purposeForAddress(dup_address);

    if (existing_purpose == "receive" &&
            (mode == NewSendingAddress || mode == EditSendingAddress)) {
        return tr(
            "Address \"%1\" already exists as a receiving address with label "
            "\"%2\" and so cannot be added as a sending address."
            ).arg(dup_address).arg(existing_label);
    }
    return tr(
        "The entered address \"%1\" is already in the address book with "
        "label \"%2\"."
        ).arg(dup_address).arg(existing_label);
}
```

The order of the numbered place markers can change when strings are translated into other languages, but each `arg` will still replace the lowest numbered unreplaced place marker, no matter where it appears. Also, if place marker `%i` appears more than once in the string, the `arg` replaces all of them.

### Translation in Designer UI Files

In Designer UI Files (`src/qt/forms/*ui`) all of `<string>` elements are subjects of translation by default. To avoid such behavior uncheck the value checkbox of the "translatable" property of the GUI element, which is not intended to be translated, or use the `notr` attribute in XML code:
```xml
<property name="placeholderText">
  <string notr="true">https://example.com/tx/%s</string>
</property>
```

## Debugging Tips

For debugging, including signal-to-slot connection issues, you can use the `QT_FATAL_WARNINGS` environment variable:

```sh
$ QT_FATAL_WARNINGS=1 src/qt/bitcoin-qt -printtoconsole -debug=qt
```

This tip can be [combined](https://github.com/bitcoin/bitcoin/pull/16118#issuecomment-503184695) with a debugger.

***

_More notes come soon..._