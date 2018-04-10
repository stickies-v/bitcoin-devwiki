https://github.com/bitcoin/bitcoin/pull/12892

A new 'label' API has been introduced for the wallet. This is intended as a
replacement for the deprecated 'account' API.

The label RPC methods mirror the account functionality, with the following functional differences:

- Labels can be set on any address, not just receiving addresses. This functionality was previously only available through the GUI. 
- Labels can be deleted by reassigning all addresses using the `setlabel` RPC method.
- There isn't support for sending transactions _from_ a label, or for determining which label a transaction was sent from.
- Labels do not have a balance.

Here are the changes to RPC methods:

| Deprecated Method       | New Method            | Notes       |
| :---------------------- | :-------------------- | :-----------|
| `getaccount`            | `getaddressinfo`      | `getaddressinfo` returns a json object with address information instead of just the name of the account as a string |
| `getaccountaddress`     | `getlabeladdress`     | `getlabeladdress` throws an error by default if the label does not already exist, but provides a `force` option for compatibility with existing applications |
| `getaddressesbyaccount` | `getaddressesbylabel` | `getaddressesbylabel` returns a json object with the addresses as keys, instead of a list of strings. |
| `getreceivedbyaccount`  | `getreceivedbylabel`  | _no change in behavior_ |
| `listaccounts`          | `listlabels`          | `listlabels` does not return a balance or accept `minconf` and `watchonly` arguments. |
| `listreceivedbyaccount` | `listreceivedbylabel` | Both methods return new `label` fields, along with `account` fields for backward compatibility. |
| `move`                  | n/a                   | _no replacement_ |
| `sendfrom`              | n/a                   | _no replacement_ |
| `setaccount`            | `setlabel`            | Both methods now: <ul><li>allow assigning labels to any address, instead of raising an error if the address is not receiving address<li>delete the previous label associated with an address, instead of making an implicit `getaccountaddress` call to ensure the previous label still has a receiving address |

| Changed Method         | Notes   |
| :--------------------- | :------ |
| `addmultisigaddress`   | `account` named parameter renamed to `label`  (`account` still accepted for backward compatibility) | 
| `getnewaddress`        | `account` named parameter renamed to `label` (`account` still accepted for backward compatibility) |
| `listunspent`          | Returns new `label` fields, along with `account` fields for backward compatibility. | 