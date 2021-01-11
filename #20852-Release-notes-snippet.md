Release notes for https://github.com/bitcoin/bitcoin/pull/20852

Done here to avoid churn in the git repo.

Notable changes
===============

Updated RPCs
------------

* The `setban` RPC can ban onion addresses again. This fixes a regression introduced in version 0.21.0.

This also:
* Breaks compatibility of the serialized banfile? I.e. opening a banfile written by this version with a previous version may drop some addresses?
* Fixes automatic misbheavior disconnect of onion outbounds.