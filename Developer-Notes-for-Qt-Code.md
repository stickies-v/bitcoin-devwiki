# Developer Notes for Qt Code

## Backward Compatibility

The source code must be compatible with the [minimum required](https://github.com/bitcoin/bitcoin/blob/master/doc/dependencies.md) Qt version which is set in the `configure.ac`:
```sh
BITCOIN_QT_CONFIGURE([5.5.1])
```

If an optional feature requires Qt higher version, or a feature was replaced by another one, use `QT_VERSION` and `QT_VERSION_CHECK` macros:

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

Do not compare `QT_VERSION` directly to Qt version of the form `0xMMNNPP` (MM = major, NN = minor, PP = patch) as such an approach is less readable and more error prone.

Every time the minimum required Qt version is bumped, `grep` all of the `QT_VERSION` instances and adjust/remove them accordingly.

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

Note that `Q_SIGNALS` and `Q_SLOTS` macros are used instead of the `signals` and `slots` keywords of the Qt `moc` (Meta-Object Compiler). It prevents potential conflicts with a 3rd party signal/slot mechanism.


***

_More notes come soon..._