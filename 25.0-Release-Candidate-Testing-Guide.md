# Testing Guide: Bitcoin Core 25.0 Release Candidate

_For feedback on this guide, please visit [#27621](https://github.com/bitcoin/bitcoin/issues/27621)_

This document outlines some of the upcoming Bitcoin Core 25.0 release changes and provides steps to help test them. This guide is meant to be the starting point for experimentation and further testing, but is in no way comprehensive! After running through the steps in this guide, you are encouraged to do your own testing.

This can be as simple as testing the same features in this guide but trying it a different way. Even better, think of features you use regularly and test that they still work as expected in the release candidate. You can also read the [release notes][release notes] to find something not covered in this guide. **This is a great way to be involved with Bitcoin's development and helps keep Bitcoin running smoothly and bug-free! Your help in this endeavor is greatly appreciated.**


## Overview

Changes covered in this testing guide include:

* Test  `-maxconnections=0` will now disable `-dnsseed` and `-listen`
* Test non-witness transaction that are greater than 65 are allowed in the mempool and relayed [#26265](https://github.com/bitcoin/bitcoin/pull/26265)
* Test Finalizing a PSBT with inputs spending Miniscript-compatible P2WSH scripts [#24149](https://github.com/bitcoin/bitcoin/pull/24149)
* Test spending coins sent to sent to P2WSH Miniscript descriptors from a descriptor wallet [#24149](https://github.com/bitcoin/bitcoin/pull/24149)

For a comprehensive list of changes in Bitcoin Core 25.0, check out the [release notes][release notes].

## Preparation

### 1. Grab Latest Release Candidate

**Current Release Candidate:** [Bitcoin Core 25.0rc2][source-code] ([release-notes][release notes])

There are two ways to grab the latest release candidate: pre-compiled binary or source code. The source code for the latest release can be grabbed from here: [latest release source code][source-code]
      
If you want to use a binary, make sure to [grab the correct one for your system][binaries].

[source-code]: https://github.com/bitcoin/bitcoin/releases/tag/v25.0rc2
[binaries]: https://bitcoincore.org/bin/bitcoin-core-25.0/test.rc2/
[release notes]: https://github.com/bitcoin-core/bitcoin-devwiki/wiki/25.0-Release-Notes-Draft

### 2. Compile Release Candidate

**If you grabbed a binary, skip this step**

Before compiling, make sure that your system has all the right [dependencies][dependencies] installed. Since descriptor wallets are now the default, it is recommended that you **compile with sqlite**(`--with-sqlite=yes`), so make sure you have installed the `sqlite3` dependency.

For more information on compiling from source, here are some guides to compile Bitcoin Core for [UNIX/Linux](https://github.com/bitcoin/bitcoin/blob/master/doc/build-unix.md), [macOS](https://github.com/bitcoin/bitcoin/blob/master/doc/build-osx.md), [Windows](https://github.com/bitcoin/bitcoin/blob/master/doc/build-windows.md), [FreeBSD](https://github.com/bitcoin/bitcoin/blob/master/doc/build-freebsd.md), [NetBSD](https://github.com/bitcoin/bitcoin/blob/master/doc/build-netbsd.md), and [OpenBSD](https://github.com/bitcoin/bitcoin/blob/master/doc/build-openbsd.md).

[dependencies]: https://github.com/bitcoin/bitcoin/blob/master/doc/dependencies.md

### 3. Setting up command line environment

First, create a temporary data directory
```shell
$ export DATA_DIR=/tmp/25-rc-test
$ mkdir $DATA_DIR
```

Next, specify the following paths. For source compiled, start from the root of your release candidate directory and run:

```shell
$ export BINARY_PATH=$(pwd)/src
```

For the downloaded binary, start from the root of the downloaded release candidate (cd ~/bitcoin-25.0rc2, for example) and run:
```shell
$ export BINARY_PATH=$(pwd)/bin
```

To avoid specifying the data directory (`-datadir=$DATA_DIR`) on each command, below are a few extra variables to set.
```shell
$ alias bitcoind-test="$(echo $BINARY_PATH)/bitcoind -datadir=$DATA_DIR"
$ alias bcli="$(echo $BINARY_PATH)/bitcoin-cli -datadir=$DATA_DIR"
```

The commands throughout the rest of the guide will look like:

```shell
$ bcli [cli args]
```


**Start node**
```shell
$  cd $DATA_DIR && echo "regtest=1" >> bitcoin.conf && cd ~
```
```shell
$  bitcoind-test
```

```shell
2023-05-11T13:42:54Z Bitcoin Core version v25.0rc2 (release build)
2023-05-11T13:42:54Z Using the 'x86_shani(1way,2way)' SHA256 implementation
2023-05-11T13:42:54Z Using RdSeed as an additional entropy source
2023-05-11T13:42:54Z Using RdRand as an additional entropy source
2023-05-11T13:42:54Z Default data directory /home/abubakarsadiq/.bitcoin
2023-05-11T13:42:54Z Using data directory /tmp/25-rc-test/regtest
2023-05-11T13:42:54Z Config file: /tmp/25-rc-test/bitcoin.conf (not found, skipping)
```
Stop the daemon
```shell
Ctrl + C
```
Look at the logs and ensure you are running the correct version.


### 4. Reset testing environment

Between sections in this guide, it's recommended to stop your node and wipe the data directory. You can use the commands provided below.

**Stop node**
```shell
$ bcli stop
```

Once the test is successful, don't forget to [stop your node and clean the datadir](#4-reset-testing-environment).

**Wipe and recreate the directory**
```shell
$ rm -r $DATA_DIR
$ mkdir $DATA_DIR
```

## Test `-maxconnections=0` will now disable `-dnsseed` and `-listen`

Add `signet=1` in the bitcoin.conf to use the signet network for testing.

```shell 
$  cd $DATA_DIR && echo "signet=1" >> bitcoin.conf && cd ~
```
```shell
$ bitcoind-test
```

If you observe the logs you will see DNS queries being logged out, which means your node is querying the DNS seeders for peers to connect to and start synchronizing the block headers. Your node is updating the chain to reach the tip.
```
2023-05-14T08:04:51Z loadblk thread exit
2023-05-14T08:04:51Z Bound to 127.0.0.1:38334
2023-05-14T08:04:51Z Bound to [::]:38333
2023-05-14T08:04:51Z Bound to 0.0.0.0:38333
2023-05-14T08:04:51Z Loaded 2 addresses from "anchors.dat"
2023-05-14T08:04:51Z 2 block-relay-only anchors will be tried for connections.
2023-05-14T08:04:51Z init message: Starting network threads…
2023-05-14T08:04:51Z net thread start
2023-05-14T08:04:51Z dnsseed thread start
2023-05-14T08:04:51Z addcon thread start
2023-05-14T08:04:51Z Loading addresses from DNS seed v7ajjeirttkbnt32wpy3c6w3emwnfr3fkla7hpxcfokr3ysd3kqtzmqd.onion:38333
2023-05-14T08:04:51Z init message: Done loading
2023-05-14T08:04:51Z msghand thread start
2023-05-14T08:04:51Z opencon thread start
2023-05-14T08:04:51Z Loading addresses from DNS seed 178.128.221.177
2023-05-14T08:04:52Z Loading addresses from DNS seed seed.signet.bitcoin.sprovoost.nl.
2023-05-14T08:04:52Z Cannot create socket for v7ajjeirttkbnt32wpy3c6w3emwnfr3fkla7hpxcfokr3ysd3kqtzmqd.onion:38333: unsupported network
2023-05-14T08:04:53Z New outbound peer connected: version: 70016, blocks=142849, peer=0 (block-relay-only)
2023-05-14T08:04:53Z 15 addresses found from DNS seeds
2023-05-14T08:04:53Z dnsseed thread exit
2023-05-14T08:04:53Z Leaving InitialBlockDownload (latching to false)
2023-05-14T08:04:54Z New outbound peer connected: version: 70016, blocks=142849, peer=1 (block-relay-only)
2023-05-14T08:04:54Z New outbound peer connected: version: 70016, blocks=142849, peer=2 (addr-fetch)
2023-05-14T08:04:54Z New outbound peer connected: version: 70016, blocks=142849, peer=3 (outbound-full-relay)
2023-05-14T08:05:02Z New outbound peer connected: version: 70016, blocks=142849, peer=4 (outbound-full-relay)
```
`Bound to 127.0.0.1:38334` and `Bound to 0.0.0.0:38333` means that Bitcoin Core is listening on the following ports and IP address:

    127.0.0.1:38334
    0.0.0.0:38333

The IP address 0.0.0.0 means that the node is listening on all available network interfaces. 

The port numbers 38333 and 38334 are the default Bitcoin Core ports (signet network) for incoming connections.


The log shows that the DNS seeders are active (`dnsseed thread start`) and trying to connect to other nodes on the network.

`"Loading addresses from DNS seed 178.128.221.177"` indicates that the node is trying to load addresses from a DNS seeder with the IP address `"178.128.221.177"`.

These seeders can help the node discover new peers on the network and facilitate the initial sync process.

At the bottom the logs shows that several outbound peers were successfully connected to the node, with varying roles such as block-relay-only, addr-fetch, and outbound-full-relay. This indicates that the node is successfully connected to the Bitcoin network and is able to send and receive information from other peers.

### Now when you set `-maxconnections=0` it will now disable `-dnsseed` and `-listen` 
 Stop your node by
```shell
Ctrl + C
```
Add `maxconnections=0` option to bitcoin.conf
```shell 
$  cd $DATA_DIR && echo "maxconnections=0" >> bitcoin.conf && cd ~
```
```shell
$ bitcoind-test
```
Observe the logs
```
2023-05-14T08:18:20Z Bitcoin Core version v25.0rc2 (release build)
2023-05-14T08:18:20Z InitParameterInteraction: parameter interaction: -connect or -maxconnections=0 set -> setting -dnsseed=0
2023-05-14T08:18:20Z InitParameterInteraction: parameter interaction: -connect or -maxconnections=0 set -> setting -listen=0
2023-05-14T08:18:20Z InitParameterInteraction: parameter interaction: -listen=0 -> setting -upnp=0
2023-05-14T08:18:20Z InitParameterInteraction: parameter interaction: -listen=0 -> setting -natpmp=0
2023-05-14T08:18:20Z InitParameterInteraction: parameter interaction: -listen=0 -> setting -discover=0
2023-05-14T08:18:20Z InitParameterInteraction: parameter interaction: -listen=0 -> setting -listenonion=0
2023-05-14T08:18:20Z InitParameterInteraction: parameter interaction: -listen=0 -> setting -i2pacceptincoming=0

```

It appears that the settings `-dnsseed` and -`listen` have been disabled as a result of setting `-maxconnections=0`.

Here are the relevant lines from the logs:
```
2023-05-14T08:18:20Z InitParameterInteraction: parameter interaction: -connect or -maxconnections=0 set -> setting -dnsseed=0
2023-05-14T08:18:20Z InitParameterInteraction: parameter interaction: -connect or -maxconnections=0 set -> setting -listen=0`
```
The logs show that the parameter interaction between `-maxconnections=0` has resulted in the disabling of `-dnsseed` and `-listen`. Specifically, `-dnsseed` has been set to 0, and -`listen` has been set to 0.

To confirm that `-dnsseed` and `-listen` have been disabled, you can check the logs for any lines that mention those settings. If they have been disabled, you should not see any lines in the logs that refer to `-dnsseed` or `-listen` after the initialization.

Try getting the list of peers in a new terminal 
```shell
$ bcli getpeerinfo
```
```
[
  
]
```
You get an empty list which means the node is not connected to any peer, and it should not be possible for any peer to connect to your node.

### The default configuration resulting from `maxconnections=0` can be overwritten by setting `-dnsseed` and `-listen`
[Clear the data directory](#4-reset-testing-environment)

Run the commands below to overwrite the default config.

```shell 
$  cd $DATA_DIR && echo "signet=1" >> bitcoin.conf && echo "maxconnections=0" && echo "dnsseed=1"  && echo "listen=1" >> bitcoin.conf>> bitcoin.conf && cd ~
```
```shell
$ bitcoind-test
```
If you observe the logs you will that
```
2023-05-14T08:33:08Z  block index            1513ms
2023-05-14T08:33:08Z Setting NODE_NETWORK on non-prune mode
2023-05-14T08:33:08Z block tree size = 142851
2023-05-14T08:33:08Z nBestHeight = 142850
2023-05-14T08:33:08Z Bound to 127.0.0.1:38334
2023-05-14T08:33:08Z Bound to [::]:38333
2023-05-14T08:33:08Z Bound to 0.0.0.0:38333
2023-05-14T08:33:08Z loadblk thread start
2023-05-14T08:33:08Z torcontrol thread start
2023-05-14T08:33:08Z Imported mempool transactions from disk: 0 succeeded, 0 failed, 0 expired, 0 already there, 0 waiting for initial broadcast
2023-05-14T08:33:08Z loadblk thread exit
2023-05-14T08:33:08Z Loaded 0 addresses from "anchors.dat"
2023-05-14T08:33:08Z 0 block-relay-only anchors will be tried for connections.
2023-05-14T08:33:08Z init message: Starting network threads…
2023-05-14T08:33:08Z net thread start
2023-05-14T08:33:08Z init message: Done loading
2023-05-14T08:33:08Z dnsseed thread start
2023-05-14T08:33:08Z msghand thread start
2023-05-14T08:33:08Z Loading addresses from DNS seed 178.128.221.177
2023-05-14T08:33:08Z addcon thread start
2023-05-14T08:33:08Z opencon thread start
2023-05-14T08:33:08Z Loading addresses from DNS seed seed.signet.bitcoin.sprovoost.nl.
2023-05-14T08:33:08Z Loading addresses from DNS seed v7ajjeirttkbnt32wpy3c6w3emwnfr3fkla7hpxcfokr3ysd3kqtzmqd.onion:38333
2023-05-14T08:33:08Z 15 addresses found from DNS seeds
2023-05-14T08:33:08Z dnsseed thread exit

```
it was overwritten and your node queries the dns seed and is listening but can not connect to a peer because maxconnections=0.

when you get the list of peers in a new terminal it will still be empty
```shell
$ bcli getpeerinfo
```
```
[
  
]
```
### Try overwriting the setting with the `maxconnections` of `2`

[Clear the data directory](#4-reset-testing-environment)

Run the commands below to add the configs.
```shell 
$  cd $DATA_DIR && echo "signet=1" >> bitcoin.conf && echo "maxconnections=2" bitcoin.conf  && cd ~
```
```shell
$ bitcoind-test -daemon
```

The list of peers will be now 2 because `maxconnections=2` will not override `-dnsssed `and `-listen`

```shell
$ bcli getpeerinfo
```
```
[
    {
    "id": 0,
    "addr": "13.213.57.217:38333",
    "addrbind": "172.20.10.2:37324",
    .......
  ,
    {
    "id": 1,
    "addr": "128.199.252.50:38333",
    "addrbind": "172.20.10.2:39204",
    .....
]
```
## Test non-witness transactions that are greater than 65 are allowed in the mempool
Tests transactions without witness data and with a size equal to or greater than 65 bytes are now accepted in the Bitcoin mempool.

To test this feature

```shell
$ bitcoind-test -regtest -daemon
```
Create a new wallet
```shell
$ bcli -regtest createwallet test-wallet
```

To test non-witness size, create a utxo.

For convienience
```shell
$ new_address=$(bcli -regtest getnewaddress)
```
Generate one block and send the subsidy output to the address you generated.
```shell
$ bcli -regtest generatetoaddress 1 $new_address
```

Generate enough blocks to make the output spendable.
```
$ bcli -regtest -generate 110
```
```
$ bcli -regtest listunspent | jq --arg new_address "$new_address" '.[] | select(.address==$new_address)'
```

```
[
  {
    "txid": "04b1a84bce28b2c989d53689404db5e3e22a2ca1875678e20286edd74dc11ace",
    "vout": 0,
  ................
```
_Note: The transaction ID and vout number are required._

To create a non-witness transaction of 64 bytes or so, You can generate an OP_RETURN output with the following dummy data in hexadecimal format: `6a00000000`.
```
Get a new change address
$ bcli -regtest getnewaddress
```
You should create a transaction with two outputs: one that sends Bitcoin to the address you generated, and the other that is an OP_RETURN output with the hex value of `6a00000000`
```
$ bcli -regtest createrawtransaction '[{"txid":"04b1a84bce28b2c989d53689404db5e3e22a2ca1875678e20286edd74dc11ace","vout":0}]' '{"data":"6a00000000"}'
```
You will get an unsigned transaction hex.


Decode the hex and verify the size is greater than 64 bytes.
```
$ bcli -regtest decoderawtransaction 0200000001ce1ac14dd7ed8602e2785687a12c2ae2e3b54d408936d589c9b228ce4ba8b1040000000000fdffffff010000000000000000076a056a0000000000000000
```
```
{
  "txid": "93b61622e84b78cb93daac2b79124159e299acef7b86d548cb518c34c27ddc99",
  "hash": "93b61622e84b78cb93daac2b79124159e299acef7b86d548cb518c34c27ddc99",
  "version": 2,
  "size": 67,
  "vsize": 67,
  "weight": 980,
  "locktime": 0,
  ...............
```
The non-witness size of the transaction is 67 bytes.

Sign the transaction.

```
$ bcli -regtest signrawtransactionwithwallet 0200000001ce1ac14dd7ed8602e2785687a12c2ae2e3b54d408936d589c9b228ce4ba8b1040000000000fdffffff010000000000000000076a056a0000000000000000
```

Try publishing it to the mempool. 

***Note: set maxfeerate to 0 to allow transaction with this huge fee to be accepted***
``` 
$ bcli -regtest sendrawtransaction 02000000000101b0ce39d2e6488b3b2bb2d449235c918e21c1e97c18b9cdaf353757d8cd4639300000000000fdffffff010000000000000000076a056a000000000247304402207fa3000e8e052eef0722e51acd601af44aab29e79b5a7b98667ec4916e80503b0220127713d14d38a10be521f79db9050c922deb866385767e5baca1bc53625e016e012102b3f766aa46f0b3fa09e78b00df29b64519c09eb7d35065fb0a5175207597ef9100000000 0
```
Observe the mempool to see your transaction.
```
$ bcli -regtest getrawmempool
```
```
[
  "33752e3ccfdc5eade58c2d44335fd7315a6087e2eaebb52ce1ec8a608b95da2a"
]
```

Generate a block to see whether it will be mined in the next block.
```
$ bcli -regtest -generate 1
```
Get the block with its hash.
```
$ bcli -regtest getblock 0fcb2b4b937e8e5f1d97443178c5f8d4b62cf0899532900234916f1e49c4f527
```

```
{
  "hash": "0fcb2b4b937e8e5f1d97443178c5f8d4b62cf0899532900234916f1e49c4f527",
  .......
  .......
  "tx": [
    "5a05b4bd25b36d65d3a4c829d2d76a6d4d4d0d448675556a691b3680a7aa29f7",
    "b38745f3f6060f55b962c353e5058d289c25435d3466e54eaa45cab645b69375"
  ]
}
```

The transaction `b38745f3f6060f55b962c353e5058d289c25435d3466e54eaa45cab645b69375`, was accepted into the mempool and included in the next block.

## Test Finalizing a PSBT with inputs spending Miniscript-compatible P2WSH scripts and test spending the coin
To test this feature, follow the steps below:

[Clear the data directory](#4-reset-testing-environment)

Add `regtest=1` in the bitcoin.conf to use the regtest for testing.

```shell 
$  cd $DATA_DIR && echo "regtest=1" >> bitcoin.conf && cd ~
```
```shell
$ bitcoind-test -daemon
```
```shell
$ bcli -regtest createwallet test-wallet
```

I have a descriptor that you will use to test this feature.

`or_d(pk(tprv8ZgxMBicQKsPerQj6m35no46amfKQdjY7AhLnmatHYXs8S4MTgeZYkWAn4edSGwwL3vkSiiGqSZQrmy5D3P5gBoqgvYP2fCUpBwbKTMTAkL/*),and_v(v:pk(tprv8ZgxMBicQKsPd3cbrKjE5GKKJLDEidhtzSSmPVtSPyoHQGL2LZw49yt9foZsN9BeiC5VqRaESUSDV2PS9w7zAVBSK6EQH3CZW9sMKxSKDwD),and_v(v:pk(tprv8iF7W37EHnVEtDr9EFeyFjQJFL6SfGby2AnZ2vQARxTQHQXy9tdzZvBBVp8a19e5vXhskczLkJ1AZjqgScqWL4FpmXVp8LLjiorcrFK63Sr),older(10))))`


Assuming `tprv8ZgxMBicQKsPerQj6m35no46amfKQdjY7AhLnmatHYXs8S4MTgeZYkWAn4edSGwwL3vkSiiGqSZQrmy5D3P5gBoqgvYP2fCUpBwbKTMTAkL` is represented as A,

`tprv8ZgxMBicQKsPd3cbrKjE5GKKJLDEidhtzSSmPVtSPyoHQGL2LZw49yt9foZsN9BeiC5VqRaESUSDV2PS9w7zAVBSK6EQH3CZW9sMKxSKDwD` is represented B and ,
`tprv8iF7W37EHnVEtDr9EFeyFjQJFL6SfGby2AnZ2vQARxTQHQXy9tdzZvBBVp8a19e5vXhskczLkJ1AZjqgScqWL4FpmXVp8LLjiorcrFK63Sr` is represented as C.

This descriptor consists of an `"or"` condition and an `"and"` condition.

scriptPubKey from the descriptor will be
```
<pk(A)> OP_CHECKSIG OP_IFDUP OP_NOTIF
  <pk(B)> OP_CHECKSIGVERIFY <pk(C)> OP_CHECKSIGVERIFY 10 OP_CHECKSEQUENCEVERIFY
OP_ENDIF
```

There are two spending paths.

1. Spending with the private key derived from the extended private key `A`.
2. Spending with the combination of private keys derived from the extended private keys `B` and `C` after a time lock of `10` blocks.

Calculate the checksum to get.
`wsh(or_d(pk(tprv8ZgxMBicQKsPerQj6m35no46amfKQdjY7AhLnmatHYXs8S4MTgeZYkWAn4edSGwwL3vkSiiGqSZQrmy5D3P5gBoqgvYP2fCUpBwbKTMTAkL/*),and_v(v:pk(tprv8ZgxMBicQKsPd3cbrKjE5GKKJLDEidhtzSSmPVtSPyoHQGL2LZw49yt9foZsN9BeiC5VqRaESUSDV2PS9w7zAVBSK6EQH3CZW9sMKxSKDwD),and_v(v:pk(tprv8iF7W37EHnVEtDr9EFeyFjQJFL6SfGby2AnZ2vQARxTQHQXy9tdzZvBBVp8a19e5vXhskczLkJ1AZjqgScqWL4FpmXVp8LLjiorcrFK63Sr),older(10)))))#xstdlsna`

Next import the descriptor.
```shell
$ bcli importdescriptors '[{"desc": "wsh(or_d(pk(tprv8ZgxMBicQKsPerQj6m35no46amfKQdjY7AhLnmatHYXs8S4MTgeZYkWAn4edSGwwL3vkSiiGqSZQrmy5D3P5gBoqgvYP2fCUpBwbKTMTAkL/*),and_v(v:pk(tprv8ZgxMBicQKsPd3cbrKjE5GKKJLDEidhtzSSmPVtSPyoHQGL2LZw49yt9foZsN9BeiC5VqRaESUSDV2PS9w7zAVBSK6EQH3CZW9sMKxSKDwD),and_v(v:pk(tprv8iF7W37EHnVEtDr9EFeyFjQJFL6SfGby2AnZ2vQARxTQHQXy9tdzZvBBVp8a19e5vXhskczLkJ1AZjqgScqWL4FpmXVp8LLjiorcrFK63Sr),older(10)))))#xstdlsna","timestamp": "now","active":true,"internal": true,"next": 0}]'
```

Derive an address from the descriptor

```shell
$ bcli deriveaddresses "wsh(or_d(pk(tprv8ZgxMBicQKsPerQj6m35no46amfKQdjY7AhLnmatHYXs8S4MTgeZYkWAn4edSGwwL3vkSiiGqSZQrmy5D3P5gBoqgvYP2fCUpBwbKTMTAkL/*),and_v(v:pk(tprv8ZgxMBicQKsPd3cbrKjE5GKKJLDEidhtzSSmPVtSPyoHQGL2LZw49yt9foZsN9BeiC5VqRaESUSDV2PS9w7zAVBSK6EQH3CZW9sMKxSKDwD),and_v(v:pk(tprv8iF7W37EHnVEtDr9EFeyFjQJFL6SfGby2AnZ2vQARxTQHQXy9tdzZvBBVp8a19e5vXhskczLkJ1AZjqgScqWL4FpmXVp8LLjiorcrFK63Sr),older(10)))))#xstdlsna" "[0,0]"
```
```shell
[
  bcrt1q2aujxv0tlu9pekg2c5jw4g0cx0ktgg99lfqy6ct6ga7kvfjksfxq7e3l7e
]
```
Generate one block to the address.
```
$ bcli generatetoaddress 1 bcrt1q2aujxv0tlu9pekg2c5jw4g0cx0ktgg99lfqy6ct6ga7kvfjksfxq7e3l7e
```
Mine 100 blocks on top to allow the output to mature.
```
$ bcli -generate 100
```
Get the list of UTXOs
```
$ bcli listunspent
```
Generate another address
```
$ bcli getnewaddress
```
Create a psbt with the utxo and the newly generated address.
```
$ bcli createpsbt '[{"txid":"41eb2bac1c8abd40a74b7c632efd555d311646bd053d0512e6cbdcebac8429f2","vout":0}]' '{"bcrt1q6v8s39fgpu2gj79nu7z4yfl5tdfc0v4xeaq73y":49.99}'
```
Process the psbt i.e append the inputs
```
$ bcli walletprocesspsbt cHNidP8BAFICAAAAAfIphKzr3MvmEgU9Bb1GFjFdVf0uY3xLp0C9ihysK+tBAAAAAAD9////AcCv9ikBAAAAFgAU0w8IlSgPFIl4s+eFUif0W1OHsqYAAAAAAAAA
```
Try finalizing the psbt

```
$ bcli finalizepsbt cHNidP8BAFICAAAAAfIphKzr3MvmEgU9Bb1GFjFdVf0uY3xLp0C9ihysK+tBAAAAAAD9////AcCv9ikBAAAAFgAU0w8IlSgPFIl4s+eFUif0W1OHsqYAAAAAAAEAjwIAAAABAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAD/////AlMA/////wIA8gUqAQAAACIAIFd5IzHr/woc2QrFJOqh+DPstCCl+kBNYXpHfWYmVoJMAAAAAAAAAAAmaiSqIant4vYcP3HR3v0/qZnfo2lTdVxpBol5mWK0i+vYNpdOjPkAAAAAAQErAPIFKgEAAAAiACBXeSMx6/8KHNkKxSTqofgz7LQgpfpATWF6R31mJlaCTAEIuAJHMEQCIDFT+Hg3kkfd06+9DQABCpQ5ZTtDkia8zrDlt0xZi4SvAiAHBDCX8c/gdFKaOomYZ8vJDHLxi9Z6QaP9bET+28Ax9gFuIQKwbR/k+0z9V6KyAjqwrksENFSHUyKohwu0SwLHRxXEi6xzZCEDTJHqWN+q1Y0RuGe+QC7ytyrQJg4/lWQOfZTaDTppSWytIQO6QdQ3kk6IPQ+MSlx2TjzKTtzX7qynVFGAiOXTlFGF5a1asmgAIgIDOM/64K4oqee2MPPeQVdTfvc63z1JCl2DmLksWeDWT4EYNypZtVQAAIABAACAAAAAgAAAAAADAAAAAA==
```
You will get the transaction hex and a `complete:true` indicating it was signed successfully.

Publish the transaction hex to the mempool
```
$ bcli sendrawtransaction 02000000000101f22984acebdccbe612053d05bd4616315d55fd2e637c4ba740bd8a1cac2beb410000000000fdffffff01c0aff62901000000160014d30f0895280f148978b3e7855227f45b5387b2a60247304402203153f878379247ddd3afbd0d00010a9439653b439226bcceb0e5b74c598b84af022007043097f1cfe074529a3a899867cbc90c72f18bd67a41a3fd6c44fedbc031f6016e2102b06d1fe4fb4cfd57a2b2023ab0ae4b043454875322a8870bb44b02c74715c48bac736421034c91ea58dfaad58d11b867be402ef2b72ad0260e3f95640e7d94da0d3a69496cad2103ba41d437924e883d0f8c4a5c764e3cca4edcd7eeaca754518088e5d3945185e5ad5ab26800000000
```
Note the transaction Id 
```
74f820e6efe429b11871468f81613c2905828b17086e2511150b1b7886a651a9
```

Mine the next block
```
$ bcli -generate 1
```
You will get the block hash and an address the subsidy was sent to.
```
{
  "address": "bcrt1q923u7h34hcqr47294n7x695409s8l0aeqf0py3",
  "blocks": [
    "4511f581428b2258cf4783225896b018dc96243551359b419ff5fc6d0c38ae29"
  ]
}

```
When you inspect the block you will see your transaction Id in the list of transactions.
```
$ bcli getblock 4511f581428b2258cf4783225896b018dc96243551359b419ff5fc6d0c38ae29
```

```
  .....
  "tx": [
    "e8df981cc24fa70019dd0a8b33354439bd70099e85c28e31c712f8a507bbfdc0",
    "74f820e6efe429b11871468f81613c2905828b17086e2511150b1b7886a651a9"
  ]

```

Thank you for your help in making Bitcoin as robust as it can be. Please remember to add a comment on [v25.0-rc2 testing issue detailing](https://github.com/bitcoin/bitcoin/issues/27621):

1. Your hardware and operating system
2. Which release candidate you tested and whether you compiled from the source or used a binary (e.g. 25.0rc2 binary or 25.0rc2 compiled from source)
3. What you tested
4. Any other relevant findings

Don't be shy about leaving a comment even if everything worked as expected! We want to hear from you and so **it doesn't count unless you leave some feedback**.

Thank you for your contribution to bitcoin core.

**Bonus tests to try later, or now :)**

##  Ensure `sendrawtransaction` rpc with transaction containing unspendable value > `maxburnamount` argument value will not be submitted.
To test this feature, follow the steps below:

[Clear the data directory](#4-reset-testing-environment)

Add `regtest=1` in the bitcoin.conf to use regtest for testing.

```shell 
$  cd $DATA_DIR && echo "regtest=1" >> bitcoin.conf && cd ~
```
```shell
$ bitcoind-test -daemon
```
```shell
$ bcli createwallet test-wallet
```
_Note: Bitcoin core do not support creating a transaction with `OP_RETURN` output whose value  is greater than zero, therefore I created an `OP_RETURN` output with a dummy hex value and value 1_

which clearly  exceeds the default `maxburnamount` of 0

The trasaction raw hex is:
```
02000000000101771af66ee30182031647c9133ebe697bbda06750d15af5e73ed26d280895a3dd0000000000fdffffff02608a0e24010000001600146a769389fb14872f826e43516200b40849a25ff500e1f50500000000116a0f48656c6c6f204f705f52657475726e0247304402204b1bdc32c9bd88a7f856555401246a314419182f27207e9b1ea9fc6eb4f20a0c02200795afa0d48c5fa2750350be75d7bca2a5241a96b0ae0ce2f1ddec1b314771370121035152789ed89e4689f6a797569cccf1f1f0a1b280bec6eb14396f3cea53881cad00000000
```
_Note: You should also use the above transaction hex string or create your own manually_
```
$ bcli decoderawtransaction 02000000000101771af66ee30182031647c9133ebe697bbda06750d15af5e73ed26d280895a3dd0000000000fdffffff02608a0e24010000001600146a769389fb14872f826e43516200b40849a25ff500e1f50500000000116a0f48656c6c6f204f705f52657475726e0247304402204b1bdc32c9bd88a7f856555401246a314419182f27207e9b1ea9fc6eb4f20a0c02200795afa0d48c5fa2750350be75d7bca2a5241a96b0ae0ce2f1ddec1b314771370121035152789ed89e4689f6a797569cccf1f1f0a1b280bec6eb14396f3cea53881cad00000000
``` 
You can the second output has a value of 1 bitcoin.
```
{
  "txid": "ff14e464aea803901e24a669f45d072974a2c11e846ab4ab87d225417a5c53c0",
  "hash": "bc6f693c51bdc6d934ae37a67e38530eed0d8baaafa94e612254350c316a77a1",
  "version": 2,
  "size": 217,
  "vsize": 136,
  "weight": 541,
  "locktime": 0,
  "vin": [
    {
      "txid": "dda39508286dd23ee7f55ad15067a0bd7b69be3e13c94716038201e36ef61a77",
      "vout": 0,
      "scriptSig": {
        "asm": "",
        "hex": ""
      },
      "txinwitness": [
        "304402204b1bdc32c9bd88a7f856555401246a314419182f27207e9b1ea9fc6eb4f20a0c02200795afa0d48c5fa2750350be75d7bca2a5241a96b0ae0ce2f1ddec1b3147713701",
        "035152789ed89e4689f6a797569cccf1f1f0a1b280bec6eb14396f3cea53881cad"
      ],
      "sequence": 4294967293
    }
  ],
  "vout": [
    {
      "value": 48.99900000,
      "n": 0,
      "scriptPubKey": {
        "asm": "0 6a769389fb14872f826e43516200b40849a25ff5",
        "desc": "addr(bcrt1qdfmf8z0mzjrjlqnwgdgkyq95ppy6yhl48dnvl4)#hkfw40dj",
        "hex": "00146a769389fb14872f826e43516200b40849a25ff5",
        "address": "bcrt1qdfmf8z0mzjrjlqnwgdgkyq95ppy6yhl48dnvl4",
        "type": "witness_v0_keyhash"
      }
    },
    {
      "value": 1.00000000,
      "n": 1,
      "scriptPubKey": {
        "asm": "OP_RETURN 48656c6c6f204f705f52657475726e",
        "desc": "raw(6a0f48656c6c6f204f705f52657475726e)#37erpn56",
        "hex": "6a0f48656c6c6f204f705f52657475726e",
        "type": "nulldata"
      }
    }
  ]
}
```

If you try sending this transaction with the default `maxburnamount` it will fail.
```
$ bcli sendrawtransaction 02000000000101771af66ee30182031647c9133ebe697bbda06750d15af5e73ed26d280895a3dd0000000000fdffffff02608a0e24010000001600146a769389fb14872f826e43516200b40849a25ff500e1f50500000000116a0f48656c6c6f204f705f52657475726e0247304402204b1bdc32c9bd88a7f856555401246a314419182f27207e9b1ea9fc6eb4f20a0c02200795afa0d48c5fa2750350be75d7bca2a5241a96b0ae0ce2f1ddec1b314771370121035152789ed89e4689f6a797569cccf1f1f0a1b280bec6eb14396f3cea53881cad00000000
```
```
error code: -25
error message:
Unspendable output exceeds maximum configured by user (maxburnamount)
```

_Note: If you try again after bumping the limit it will fail because the transaction id is not recognized or does not exist within your node's context_


## Test `unloadwallet` fails if a rescan is in progress 
[#26618](https://github.com/bitcoin/bitcoin/pull/26618)

[Clear the data directory](#4-reset-testing-environment)

Add `signet=1` in the bitcoin.conf to use the signet for testing.

```shell 
$  cd $DATA_DIR && echo "signet=1" >> bitcoin.conf && cd ~
```
```shell
$ bitcoind-test -daemon
```
```shell
$ bcli -regtest createwallet test-wallet
```
Import a descritor wallet which will trigger a rescan.

use your own descriptor or you can use this one `wpkh(tprv8gxXKvjGCb1uidca4SXuUJJdK23mBBxtZUT91vGMLgJEEuxF35FHHCnCwqoFjTgL7Gb1AK8cgwCgy3TDq8oef5oDAFzssDiiVm3bDLcgLau/0/*)#9uzd3wgm`


The parameters we will used
- "desc":Descriptor to import.
-"active": If true the descriptor will be the active descriptor for the corresponding output type/externality
- "internal": A boolean value Whether matching outputs should be treated as range wil be set
-"keypool": If true the default keypool range will be used.

```
$ bcli importdescriptors '[{"desc":"wpkh(tprv8gxXKvjGCb1uidca4SXuUJJdK23mBBxtZUT91vGMLgJEEuxF35FHHCnCwqoFjTgL7Gb1AK8cgwCgy3TDq8oef5oDAFzssDiiVm3bDLcgLau/0/*)#9uzd3wgm", "timestamp": 0,"active": false,"internal": true,"keypool":true}]'
```
While it is importing open another terminal window and try unloading the wallet.

```
$ bcli unloadwallet test-wallet
```

The wallet will not be unloaded it will return an error message
```
error code: -4
error message:
Wallet is currently rescanning. Abort existing rescan or wait.
```

## Test Command specified by `shutdownnotify` are executed synchronously before shutdown

[#23395](https://github.com/bitcoin/bitcoin/pull/23395) 

To test this feature, follow the steps below:

[Clear the data directory](#4-reset-testing-environment)

Add `regtest=1` in the bitcoin.conf to use the regtest for testing.

```shell 
$  cd $DATA_DIR && echo "regtest=1" >> bitcoin.conf && cd ~
```
```shell
$ bitcoind-test -daemon -shutdownnotify="bcli touch hello.txt"
```
```shell
$ bcli stop
```
```shell
$ ls
```

And you will see `hello.txt` in the lists of files in the directory.

## Restrict Address Purposes strings to known values 'send', 'receive', and 'refund'. Unrecognized purposes trigger loading warnings and errors in listlabels RPC

To test this feature, follow the steps below:

[Clear the data directory](#4-reset-testing-environment)

Add `regtest=1` in the bitcoin.conf to use the regtest for testing.

```shell 
$  cd $DATA_DIR && echo "regtest=1" >> bitcoin.conf && cd ~
```
```shell
$ bitcoind-test -daemon
```
```shell
$ bcli -regtest createwallet test-wallet
```

When you try listlabels with any argument other than  'send', 'receive', and 'refund' you get an error.

```
$ bcli listlabels invalid
```
```
error code: -8
error message:
Invalid 'purpose' argument, must be a known purpose string, typically 'send', or 'receive'.
```
## Test `listunspentwith` `include_immature_coinbase` include coinbase UTXOs that don't meet the minimum spendability depth requirement
[#25730](https://github.com/bitcoin/bitcoin/pull/25730)

To test this feature, follow the steps below:

[Clear the data directory](#4-reset-testing-environment)

Add `regtest=1` in the bitcoin.conf to use the regtest for testing.

```shell 
$  cd $DATA_DIR && echo "regtest=1" >> bitcoin.conf && cd ~
```
```shell
$ bitcoind-test -daemon
```
```shell
$ bcli -regtest createwallet test-wallet
```
Mine 1 block
```
$ bcli -generate 1
```
```
$ bcli listunspent 
```
```
[
  
]
```

You expect to see one output in the list of unspent transactions. However, you might not find it there if the output is from a coinbase transaction because it is unspendable until it has reached a depth of 100 blocks

You can view the output by calling the RPC with the `include_immature_coinbase=true` query option, but keep in mind that it is still unspendable until it has matured.
```
$ bcli -named listunspent query_options='{ "include_immature_coinbase":true }'
```
```
[
  {
    "txid": "c5650762db3c5f38351cf85c18c1f3e65d3161ba5c297a994627dfec68c18f4f",
    "vout": 0,
    "address": "bcrt1qe86cj0n00y7f2mm7a46346nddfv53yckpt9mgl",
    "label": "",
    "scriptPubKey": "0014c9f5893e6f793c956f7eed751aea6d6a59489316",
    "amount": 50.00000000,
    "confirmations": 1,
    "spendable": true,
    "solvable": true,
    "desc": "wpkh([e1842a94/84'/1'/0'/0/0]02ce315b26c7cd860297093a5fe0ae31379c5ad118385ceec2b523dee19dab5fae)#5ywzv2y0",
    "parent_descs": [
      "wpkh(tpubD6NzVbkrYhZ4Yk7wbT1tRJAg5rQakNnmWPxMTzk5wmwCghzFGVV13Kz3WpddivixHi9iVXgBKy7LPG3EppXacsdufosyCT2tctaHpWc2zqu/84'/1'/0'/0/*)#7ttv4pe5"
    ],
    "safe": true
  }
]

```
## Test the new endpoint /rest/deploymentinfo that has been added for fetching various state info regarding deployments of consensus changes.
[#25412](https://github.com/bitcoin/bitcoin/pull/25412)

[Clear the data directory](#4-reset-testing-environment)

Add `signet=1` in the bitcoin.conf to use the signet for testing.

```shell 
$  cd $DATA_DIR && echo "signet=1" >> bitcoin.conf && echo "rest=1" >> bitcoin.conf && cd ~
```
```shell
$ bitcoind-test -daemon
```
Run this command to get the list of all the new consensus changes represend by their bip number

```shell
$ curl localhost:38332/rest/deploymentinfo.json | json_pp
```

```
{
   "deployments" : {
      "bip34" : {
         "active" : true,
         "height" : 1,
         "type" : "buried"
      },
      "bip65" : {
         "active" : true,
         "height" : 1,
         "type" : "buried"
      },
      .......

```
The deployments field contains a list of consensus changes, identified by their BIP numbers.
Each consensus change includes information such as its status (active or inactive), height (the block height at which it was activated), and type (buried, locked-in, or active).

## Ensure `verifychain` returns false if the check can't be completed
[#25574](https://github.com/bitcoin/bitcoin/pull/25574)

To test this feature, follow the steps below:

[Clear the data directory](#4-reset-testing-environment)

After clearing the data directory and creating a new one, add the following configurations to the bitcoin.conf file.

`signet=1` we are using the signet network to test.

`prune=550` we will pruned the blockchain to make `verifychain` fail due to missing blocks.

```shell
$ cd $DATA_DIR && echo "signet=1" >> bitcoin.conf && echo "prune=550" >> bitcoin.conf  && cd ~
```
```shell
$ bitcoind-test -daemon
```
Leave the daemon for some minutes to finish syncing the chain.

You can check the progress.

```shell
$ bcli -getinfo
```
```
Chain: signet
Blocks: 142573
Headers: 142573
Verification progress: 99.9999%
Difficulty: 0.003064903846153846

Network: in 0, out 10, total 10
Version: 250000
Time offset (s): -22
Proxies: n/a
Min tx relay fee rate (BTC/kvB): 0.00001000

Warnings: (none)
```
The daemon has finished syncing, pruned at 450 blocks.

Next step is to try `verifychain` rpc with the below parameters.

`checklevel`=1, `nblocks`=0

checklevel=4 means we want up to 1 level verification, for all the blocks.


- level 0 reads the blocks from disk

- level 1 verifies block validity

Each level includes the checks of the previous levels

`nblocks`=0 means verify blocks from genesis up to the chain tip.

```shell
$ bcli verifychain 1 0
```
The output should be
```
false
```
because the blockchain was pruned and some blocks' data was missing, making it impossible to verify.

## Ensure with `-checkblocks` or `-checklevel` options Bitcoin Core will return an error at startup if the verification checks cannot be completed due to an insufficient dbcache

[#25574](https://github.com/bitcoin/bitcoin/pull/25574)

_There will be no need to clear the data directory for this test, use the signet network, pruned._

Stop the node and add the options in the bitcoin.conf.

`dbcache=20` would allocate 20 megabytes of memory for database caching.

`checklevel=1` we want 1 level verification.

level 2 verifies undo data

level 3 checks disconnection of tip blocks

Each level includes the checks of the previous levels

`checkblocks=0` we want all the blocks up to the tip to be verified.

Then run the following command
```shell
$ cd $DATA_DIR && echo "dbcache=20" >> bitcoin.conf && echo "checklevel=3" >> bitcoin.conf && echo "checkblocks=0" >> bitcoin.conf && cd ~
```
```
$ bitcoind-test
```
At the bottom of the log you can see
```
2023-05-12T10:09:43Z Verifying last 142577 blocks at level 1
2023-05-12T10:09:43Z Verification progress: 0%
2023-05-12T10:10:01Z Verification progress: 10%
2023-05-12T10:10:30Z Verification progress: 20%
2023-05-12T10:10:42Z VerifyDB(): block verification stopping at height 75904 (no data). This could be due to pruning or use of an assumeutxo snapshot.
2023-05-12T10:10:42Z Skipped verification of level >=1 (insufficient database cache size). Consider increasing -dbcache.
2023-05-12T10:10:42Z Verification: No coin database inconsistencies in last 66673 blocks (102270 transactions)
2023-05-12T10:10:42Z Error: Insufficient dbcache for block verification
Error: Insufficient dbcache for block verification
2023-05-12T10:10:42Z Shutdown: In progress...
2023-05-12T10:10:42Z scheduler thread exit
2023-05-12T10:10:42Z Shutdown: done

```

The dbcache is insufficient to finish the verification.


## Test how to verify binaries after [#27440](https://github.com/bitcoin/bitcoin/27440) Update 

Browse the list of [frequent builder-keys](https://github.com/bitcoin-core/guix.sigs/tree/main/builder-keys) and decide which keys you want to trust.


Download any gpg public key there

e.g. You downloaded Josibake pgp public key.

Open a terminal or command prompt.

Navigate to the directory where you downloaded the josibake.gpg file.

Run the following command to import the key:
```shell
$ gpg --import josibake.gpg
```
This command imports the key into your GPG keychain.

Verify Bitcoin Core v25rc2:

a. If you have the Bitcoin Core source code, navigate to the root directory of the source code. 

```shell
$ cd /path/to/bitcoin-core

```
Make sure to replace /path/to/bitcoin-core with the actual path to the Bitcoin Core source code 

Run the command
```
$ ./contrib/verify-binaries/verify.py --trusted-keys josibake.gpg pub 25.0-rc2
```

b. If you are using the pre-built binaries, download the verify.py script from the [Bitcoin Core repository](https://github.com/bitcoin/bitcoin/blob/master/contrib/verify-binaries/verify.py)



Run the command
```shell
$ ./verify.py --trusted-keys josibake.gpg pub 25.0-rc2
```

When you run the verification script, it will perform the verification process and display the results directly in the terminal. The output will include information about signature verification, hash verification, and any warnings or errors encountered during the process of verifying the downloaded binaries.

Please note that the verification process may take some time.

By following these steps, you can verify the Bitcoin Core v25rc2 binaries using the provided public key and the verification script


Kudos if you make it this far 👏🎉

Thanks for your contribution and for taking the time to make Bitcoin awesome. For feedback on this guide, please visit [#27621](https://github.com/bitcoin/bitcoin/issues/27621)