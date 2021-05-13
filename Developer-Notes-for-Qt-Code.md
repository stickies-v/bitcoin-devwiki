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

1. Log into [GitHub](https://github.com), navigate to the [GUI repo](https://github.com/bitcoin-core/gui), and [fork](https://docs.github.com/en/github/getting-started-with-github/fork-a-repo) it.

2. Clone the GUI repo locally:
```sh
$ git clone https://github.com/bitcoin-core/gui.git && cd gui
```

3. Set up your forked repo as a local git remote (_"satoshi"_ is used as an example GitHub username)::
```sh
$ git remote add satoshi https://github.com/satoshi/gui.git
$ git config remote.pushDefault satoshi
```

4. Verify steps were performed correctly:
```sh
$ git remote -v
origin	https://github.com/bitcoin-core/gui.git (fetch)
origin	https://github.com/bitcoin-core/gui.git (push)
satoshi	https://github.com/satoshi/gui.git (fetch)
satoshi	https://github.com/satoshi/gui.git (push)
```

5. To synchronize your local and forked repos:
```sh
$ git switch master
$ git pull --ff-only
$ git push
```

### Reviewing Pull Request

The following highlights important steps of the review process using [PR 42](https://github.com/bitcoin-core/gui/pull/42) as an example.

#### Fetch PR branch

It is recommended to review pull requests (PR) locally as this allows you to build binaries and test functionality.

To fetch the PR branch to your local repo, run the following command:
```sh
$ git fetch origin pull/42/head:pr42.01 && git switch pr42.01
```

This command clones [PR 42](https://github.com/bitcoin-core/gui/pull/42) into a new branch named `pr42.01` then switches to it. This example branch name includes a sequence number `.01`. Sequence numbers make it convenient to compare how the PR changes as development occurs.

#### PR is updated

As a PR goes through the review process, changes are made. When changes occur, you must re-fetch the PR to test it locally. It is recommended to use separate local branches for every fetch. Each fetch should increment the sequence number used.

To examine differences between updates:
```sh
$ git diff pr42.03 pr42.04
```

To examine differences when a PR is rebased:
```sh
$ git range-diff master pr42.03 pr42.04
```

#### PR is merged

You can delete all local branches of the PR after it has been successfully merged:
```sh
$ git switch master
$ git branch -D $(git branch | grep pr42)
```

## Backward Compatibility

The source code must always maintain compatibility with the [minimum required](https://github.com/bitcoin/bitcoin/blob/master/doc/dependencies.md) Qt version, which is set in `configure.ac`:
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

When subclassing the `QObject` class follow the pattern below:

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

It is assumed that developers follow these guides:
- [Translation Process](https://github.com/bitcoin/bitcoin/blob/master/doc/translation_process.md)
- [Translation Strings Policy](https://github.com/bitcoin/bitcoin/blob/master/doc/translation_strings_policy.md)

### Translation in C++ Code

#### String Literals

To translate a string literal (quoted text), use the [`QObject::tr`](https://doc.qt.io/qt-5.12/qobject.html#tr) function which returns a translated `QString` object:

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

#### Translator Comments

It is highly recommended to provide a translation context via [translator comments](https://doc.qt.io/qt-5/i18n-source-translation.html#translator-comments) to every translatable string:
```cpp
void BitcoinGUI::setNetworkActive(bool network_active)
{
    updateNetworkState();
    m_network_context_menu->clear();
    m_network_context_menu->addAction(
        //: A context menu item. The "Peers tab" is an element of the "Node window".
        tr("Show Peers tab"),
        [this] {
            rpcConsole->setTabFocus(RPCConsole::TabTypes::PEERS);
            showDebugWindow();
        });
    m_network_context_menu->addAction(
        network_active ?
            //: A context menu item.
            tr("Disable network activity") :
            //: A context menu item. The network activity was disabled previously.
            tr("Enable network activity"),
        [this, new_state = !network_active] { m_node.setNetworkActive(new_state); });
}
```

#### Substrings

Some substrings are unknown at compile-time or are not intended to be translated, e.g., error messages, command line options, default file names, etc. In such cases, it is recommended to:
- avoid unwanted translation
- not break up the whole string into fragments that are translated separately

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

The order of the numbered place markers can be changed when strings are translated into other languages, but each `arg` will still replace the lowest-numbered unreplaced place marker, no matter where it appears. Also, if place marker `%i` appears more than once in the string, the `arg` replaces all instances.

### Translation in Designer UI Files

In Designer UI Files (`src/qt/forms/*ui`) all `<string>` elements are subject to translation by default. To avoid such behavior uncheck the "translatable" property checkbox of the GUI element which is not intended to be translated. Alternatively, use the `notr` attribute in XML code:
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