New RPCs
--------

- The `scanblocks` RPC returns the relevant blockhashes from a set of descriptors by
  scanning all blockfilters in the given range. It can be used in combination with
  the `getblockheader` and `rescanblockchain` RPCs to achieve fast wallet rescans.
  Note that this functionality can only be used if a compact block filter index
  (`-blockfilterindex=1`) has been constructed by the node. (#23549)