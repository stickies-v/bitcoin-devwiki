### Contents

1. [Backward Compatibility](#backward-compatibility)
2. [`QObject` Subclassing Style](#qobject-subclassing-style)
3. [Debugging Tips](#debugging-tips)

## Backward Compatibility

The source code must be compatible with the [minimum required](https://github.com/bitcoin/bitcoin/blob/master/doc/dependencies.md) Qt version which is set in `configure.ac`:
```sh
BITCOIN_QT_CONFIGURE([5.5.1])
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

For debugging, including signal to slot connection issues, you can use the `QT_FATAL_WARNINGS` environment variable:

```sh
$ QT_FATAL_WARNINGS=1 src/qt/bitcoin-qt -printtoconsole -debug=qt
```

This tip can be [combined](https://github.com/bitcoin/bitcoin/pull/16118#issuecomment-503184695) with a debugger.

***

_More notes come soon..._