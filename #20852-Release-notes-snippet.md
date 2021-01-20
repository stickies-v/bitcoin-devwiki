Release notes for https://github.com/bitcoin/bitcoin/pull/20852 and https://github.com/bitcoin/bitcoin/pull/20966

Done here to avoid churn in the git repo.

Notable changes
===============

Updated RPCs
------------

* The `setban` RPC can ban onion addresses again. This fixes a regression introduced in version 0.21.0. The stored onion bans in `banlist.dat` by version 0.21.1 will be ignored by previous versions and treated invalid. (#20852)

Files
-----

- The list of banned hosts and networks (via `setban` RPC) is now saved on disk
  in JSON format in `banlist.json` instead of `banlist.dat`. Both files are
  read on startup and contents merged. Updateds are only written to the new
  `banlist.json`. A future version of Bitcoin Core may completely ignore
  `banlist.dat`. (#20966)