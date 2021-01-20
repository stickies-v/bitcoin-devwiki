Release notes for https://github.com/bitcoin/bitcoin/pull/20852 and https://github.com/bitcoin/bitcoin/pull/20966

Done here to avoid churn in the git repo.

Notable changes
===============

Updated RPCs
------------

* The `setban` RPC can ban onion addresses again. This fixes a regression introduced in version 0.21.0. The stored onion bans in `banlist.dat` by version 0.21.1 will be ignored by previous versions and treated invalid. (#20852)