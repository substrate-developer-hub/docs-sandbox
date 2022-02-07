---
title: Start a Private Network
slug: /tutorials/v3/private-network
hideNav: true
version: '3.0'
sideNav: privateNetwork
section: tutorials
category: private network
keywords: permissioned, consortium, authority, network
difficulty: 1
duration: 2 Hour
relevantSkills:
  - Rust
  - Blockchain basics
  - FRAME
---

## Introduction

In this tutorial we will learn and practice how to start a blockchain network with a
validator/authority set of your choosing using Substrate.

You should already have version `v3.0.0` of the
[Substrate Node Template](https://github.com/substrate-developer-hub/substrate-node-template)
compiled on your computer from when you completed the
[Create Your First Substrate Chain Tutorial](/tutorials/v3/create-your-first-substrate-chain).
If you do not, please complete that tutorial.

<Message
  type={`yellow`}
  title={`Information`}
  text="Experienced developers who truly prefer to skip that tutorial, you may install the node template 
      according to the instructions in its readme.
      "
/>

## What You Will be Doing

Before we even get started, let's layout what we're going to do over the course of this tutorial. We will:

<TutorialObjective
  data={{
    textLineOne: '1. Have Alice and Bob start a blockchain',
    url: '/tutorials/v3/private-network#alice-and-bob-start-a-blockchain',
  }}
/>
<TutorialObjective
  data={{
    textLineOne:
      '2. Generate ed25519 and sr25519 key-pairs for use as a network authority.',
    url: '#generate-your-own-keys',
  }}
/>
<TutorialObjective
  data={{
    textLineOne: '3. Create and edit a chain spec JSON file.',
    url: '#create-a-custom-chain-spec',
  }}
/>
<TutorialObjective
  data={{
    textLineOne: '4. Launch your custom chain.',
    url: '#launch-your-private-network',
  }}
/>

## Learning outcomes

- Learn how to create authority nodes for a private network
- Learn how to configure a JSON chain spec file

## Alice and Bob start a blockchain

Before we generate our own keys, and start a truly unique Substrate network, let's learn the
fundamentals by starting with a pre-defined network specification called `local` with two
pre-defined (and definitely not private!) keys known as Alice and Bob.
**This portion of the tutorial should be run on a single workstation with a single Substrate binary.**

### Alice Starts First

Alice (or whomever is playing her) should run these commands from node-template repository root.

<Message
  type={`yellow`}
  title={`Information`}
  text="Here we\'ve explicitly shown the `purge-chain` command. In the future we will omit this You should
    purge old chain data any time you\'re trying to start a new network.
    "
/>

```bash
# Purge any chain data from previous runs
# You will be prompted to type `y`
./target/release/node-template purge-chain --base-path /tmp/alice --chain local
```

```bash
# Start Alice's node
./target/release/node-template \
  --base-path /tmp/alice \
  --chain local \
  --alice \
  --port 30333 \
  --ws-port 9945 \
  --rpc-port 9933 \
  --node-key 0000000000000000000000000000000000000000000000000000000000000001 \
  --telemetry-url 'wss://telemetry.polkadot.io/submit/ 0' \
  --validator
```

Let's look at those flags in detail:

| <div style="min-width:110pt"> Flags </div> | Descriptions                                                                                                                                                                                                                                                                                                                               |
| ------------------------------------------ | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `--base-path`                              | Specifies a directory where Substrate should store all the data related to this chain. If this value is not specified, a default path will be used. If the directory does not exist it will be created for you. If other blockchain data already exists there you will get an error. Either clear the directory or choose a different one. |
| `--chain local`                            | Specifies which chain specification to use. There are a few prepackaged options including `local`, `development`, and `staging` but generally one specifies their own chain spec file. We'll specify our own file in a later step.                                                                                                         |
| `--alice`                                  | Puts the predefined Alice keys (both for block production and finalization) in the node's keystore. Generally one should generate their own keys and insert them with an RPC call. We'll generate our own keys in a later step. This flag also makes Alice a validator.                                                                    |
| `--port 30333`                             | Specifies the port that your node will listen for p2p traffic on. `30333` is the default and this flag can be omitted if you're happy with the default. If Bob's node will run on the same physical system, you will need to explicitly specify a different port for it.                                                                   |
| `--ws-port 9945`                           | Specifies the port that your node will listen for incoming WebSocket traffic on. The default value is `9944`. This example uses a custom web socket port number (`9945`).                                                                                                                                                                  |
| `--rpc-port 9933`                          | Specifies the port that your node will listen for incoming RPC traffic on. `9933` is the default, so this parameter may be omitted.                                                                                                                                                                                                        |
| `--node-key <key>`                         | The Ed25519 secret key to use for `libp2p` networking. The value is parsed as a hex-encoded Ed25519 32 byte secret key, i.e. 64 hex characters. WARNING: Secrets provided as command-line arguments are easily exposed. Use of this option should be limited to development and testing.                                                   |
| `--telemetry-url`                          | Tells the node to send telemetry data to a particular server. The one we've chosen here is hosted by Parity and is available for anyone to use. You may also host your own (beyond the scope of this article) or omit this flag entirely.                                                                                                  |
| `--validator`                              | Means that we want to participate in block production and finalization rather than just sync the network.                                                                                                                                                                                                                                  |

When the node starts, you should see output similar to this:

```
2021-03-10 17:34:27  Substrate Node
2021-03-10 17:34:27  ✌️  version 3.0.0-1c5b984-x86_64-linux-gnu
2021-03-10 17:34:27  ❤️  by Substrate DevHub <https://github.com/substrate-developer-hub>, 2017-2021
2021-03-10 17:34:27  📋 Chain specification: Local Testnet
2021-03-10 17:34:27  🏷 Node name: Alice
2021-03-10 17:34:27  👤 Role: AUTHORITY
2021-03-10 17:34:27  💾 Database: RocksDb at /tmp/alice/chains/local_testnet/db
2021-03-10 17:34:27  ⛓  Native runtime: node-template-100 (node-template-1.tx1.au1)
2021-03-10 17:34:27  🔨 Initializing Genesis block/state (state: 0xea47…9ba8, header-hash: 0x9d07…7cce)
2021-03-10 17:34:27  👴 Loading GRANDPA authority set from genesis on what appears to be first startup.
2021-03-10 17:34:27  ⏱  Loaded block-time = 6000 milliseconds from genesis on first-launch
2021-03-10 17:34:27  Using default protocol ID "sup" because none is configured in the chain specs
2021-03-10 17:34:27  🏷 Local node identity is: 12D3KooWEyoppNCUx8Yx66oV9fJnriXwCcXwDDUA2kj6vnc6iDEp
2021-03-10 17:34:27  📦 Highest known block at #0
2021-03-10 17:34:27  〽️ Prometheus server started at 127.0.0.1:9615
2021-03-10 17:34:27  Listening for new connections on 127.0.0.1:9945.
2021-03-10 17:34:32  💤 Idle (0 peers), best: #0 (0x9d07…7cce), finalized #0 (0x9d07…7cce), ⬇ 0 ⬆ 0
2021-03-10 17:34:37  💤 Idle (0 peers), best: #0 (0x9d07…7cce), finalized #0 (0x9d07…7cce), ⬇ 0 ⬆ 0
...
```

A few things to notice:

- `🔨 Initializing Genesis block/state (state: 0xea47…9ba8, header-hash: 0x9d07…7cce)` tells which
  genesis block the node is using. When you start the next node, verify that these values are
  equal.
- `🏷 Local node identity is: 12D3KooWEyoppNCUx8Yx66oV9fJnriXwCcXwDDUA2kj6vnc6iDEp` shows the
  Peer ID that Bob will need when booting from Alice's node. This value was determined by the
  `--node-key` that was used to start Alice's node.
- No blocks are being produced yet. Blocks will start being produced once another
  node joins the network.

More details about all of these flags and others that I haven't mentioned are available by running
`./target/release/node-template --help`.

### Attach a UI

You can tell a lot about your node by watching the output it produces in your terminal. There is
also a nice graphical user interface called Polkadot-JS Apps, or just "Apps" for short.

In your web browser, navigate to
[https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9945#/explorer](https://polkadot.js.org/apps/?rpc=ws%3A%2F%2F127.0.0.1%3A9945#/explorer).

<Message
  type={`green`}
  title={`Tip`}
  text="Some Ad blockers (e.g. the built-in Shield in Brave browser, pihole, browser extensions, etc.)
	  block the connection to a local node. In case you have any Ad blockers in your browser, make
	  sure to check them and turn them off as needed if you have issues connecting to your local node. 
	  Also some browsers, notably Firefox, will not connect to a local node from a https website.
	  An easy work around is to try another browser, like Chromium. Alternatively you can
    [host this interface locally](https://github.com/polkadot-js/apps#development).
    "
/>

The link provided above includes the `rpc` URL parameter, which instructs the Apps UI to connect to
the URL that was provided as its value (in this case, your local node). To manually configure Apps
UI to connect to another node:

- Click on the top left network icon

  ![Top Left Network Icon](../img/tutorials/05-private-network/private-network-top-left-network-icon.png)

- A popup dialog appears. Expand **DEVELOPMENT** and ensure the custom endpoint is set to
  `ws://127.0.0.1:9945`.

  ![Select Network](../img/tutorials/05-private-network/private-network-select-network.png)

- To connect to a custom node and port, you just need to specify the endpoint by choosing
  `custom endpoint` and type in your own endpoint. In this way you can use a single instance of Apps
  UI to connect to various nodes. Click **Switch** icon to actually switch to the new endpoint when
  necessary.

  ![Custom Endpoint](../img/tutorials/05-private-network/private-network-custom-endpoint.png)

You should now see something like this example from the **Network** and **Explorer** page.

![No blocks in polkadot-js-apps](../img/tutorials/05-private-network/private-network-no-blocks.png)

<Message
  type={`gray`}
  title={`Note`}
  text="If you do not want to run your hosted version of Polkadot-JS Apps UI while connecting to Substrate 
    node you have deployed remotely, you can [configure SSH local port forwarding](https://www.booleanworld.com/guide-ssh-port-forwarding-tunnelling/)
    to forward local request to the `ws-port` listened by the remote host.
    "
/>

### Bob Joins

Now that Alice's node is up and running, Bob can join the network by bootstrapping from her node.
His command will look very similar.

```bash
./target/release/node-template purge-chain --base-path /tmp/bob --chain local
```

```bash
./target/release/node-template \
  --base-path /tmp/bob \
  --chain local \
  --bob \
  --port 30334 \
  --ws-port 9946 \
  --rpc-port 9934 \
  --telemetry-url 'wss://telemetry.polkadot.io/submit/ 0' \
  --validator \
  --bootnodes /ip4/127.0.0.1/tcp/30333/p2p/12D3KooWEyoppNCUx8Yx66oV9fJnriXwCcXwDDUA2kj6vnc6iDEp
```

Most of these options are already explained above, but there are a few points worth mentioning.

- Because these two nodes are running on the same physical machine, Bob must specify different
  `--base-path`, `--port`, `--ws-port`, and `--rpc-port` values.
- Bob has added the `--bootnodes` flag and specified a single boot node, namely Alice's. He must
  correctly specify these three pieces of information which Alice can supply for him.
  - Alice's IP Address, probably `127.0.0.1`
  - Alice's Port, she specified `30333`
  - Alice's Peer ID, copied from her log output.

If all is going well, after a few seconds, the nodes should peer together and start producing
blocks. You should see some lines like the following in the console that started Alice node:

```
...
2021-03-10 17:47:32  🔍 Discovered new external address for our node: /ip4/127.0.0.1/tcp/30333/p2p/12D3KooWEyoppNCUx8Yx66oV9fJnriXwCcXwDDUA2kj6vnc6iDEp
2021-03-10 17:47:32  🔍 Discovered new external address for our node: /ip4/<your computer's LAN IP>/tcp/30333/p2p/12D3KooWEyoppNCUx8Yx66oV9fJnriXwCcXwDDUA2kj6vnc6iDEp
2021-03-10 17:47:33  💤 Idle (1 peers), best: #0 (0x9d07…7cce), finalized #0 (0x9d07…7cce), ⬇ 1.0kiB/s ⬆ 1.0kiB/s
2021-03-10 17:47:36  🙌 Starting consensus session on top of parent 0x9d07d1757a9ca248e58141ce52a11fca37f71007dec16650b87a853f0d4c7cce
2021-03-10 17:47:36  🎁 Prepared block for proposing at 1 [hash: 0x727826a5e6fba9a13af11422d4677b5f0743cc733c382232328e69fd307d1d2f; parent_hash: 0x9d07…7cce; extrinsics (1): [0x768a…a9e2]]
2021-03-10 17:47:36  🔖 Pre-sealed block for proposal at 1. Hash now 0x4841d8b2e62483fa4702b3ddcd1b603803842374dcdc1e9533ad407708b33dd8, previously 0x727826a5e6fba9a13af11422d4677b5f0743cc733c382232328e69fd307d1d2f.
2021-03-10 17:47:36  ✨ Imported #1 (0x4841…3dd8)
2021-03-10 17:47:36  ✨ Imported #1 (0xb241…2ae8)
2021-03-10 17:47:38  💤 Idle (1 peers), best: #1 (0x4841…3dd8), finalized #0 (0x9d07…7cce), ⬇ 0.8kiB/s ⬆ 0.8kiB/s
2021-03-10 17:47:42  ♻️  Reorg on #1,0x4841…3dd8 to #2,0x8b6a…dce6, common ancestor #0,0x9d07…7cce
2021-03-10 17:47:42  ✨ Imported #2 (0x8b6a…dce6)
2021-03-10 17:47:43  💤 Idle (1 peers), best: #2 (0x8b6a…dce6), finalized #0 (0x9d07…7cce), ⬇ 0.8kiB/s ⬆ 0.7kiB/s
2021-03-10 17:47:48  🙌 Starting consensus session on top of parent 0x8b6a3ab2fe9891b1af008ea0d92dae9bc84cfa5578231e81066d47928822dce6
2021-03-10 17:47:48  🎁 Prepared block for proposing at 3 [hash: 0xb887aef2097eff5869e38ccec0302bce372ad05ac2cdf9cc4725c38ec071fb7a; parent_hash: 0x8b6a…dce6; extrinsics (1): [0x82ac…2f20]]
2021-03-10 17:47:48  🔖 Pre-sealed block for proposal at 3. Hash now 0x34d608dd8be6b82bef4a7aaae1ec80930a5c4b8cf9bdc99013410e91544f3a2a, previously 0xb887aef2097eff5869e38ccec0302bce372ad05ac2cdf9cc4725c38ec071fb7a.
2021-03-10 17:47:48  ✨ Imported #3 (0x34d6…3a2a)
2021-03-10 17:47:48  💤 Idle (1 peers), best: #3 (0x34d6…3a2a), finalized #0 (0x9d07…7cce), ⬇ 0.7kiB/s ⬆ 0.8kiB/s
2021-03-10 17:47:53  💤 Idle (1 peers), best: #3 (0x34d6…3a2a), finalized #1 (0xb241…2ae8), ⬇ 0.6kiB/s ⬆ 0.7kiB/s
2021-03-10 17:47:54  ✨ Imported #4 (0x2b8a…fdc4)
2021-03-10 17:47:58  💤 Idle (1 peers), best: #4 (0x2b8a…fdc4), finalized #2 (0x8b6a…dce6), ⬇ 0.7kiB/s ⬆ 0.6kiB/s
...
```

These lines shows that Bob has peered with Alice (**`1 peers`**), they have produced some blocks
(**`best: #4 (0x2b8a…fdc4)`**), and blocks are being finalized (**`finalized #2 (0x8b6a…dce6)`**).

Looking at the console that started Bob's node, you should see something similar.

Once you've verified that both nodes are running as expected, you should shut them down. Although
this is not strictly required, so long as you don't have conflicting ports and the same chain spec.
The next section of this tutorial will include commands to restart new nodes when necessary.

## Generate your own keys

Now that we know the fundamentals and the command line options, it's time to generate our own keys
rather than using the well-known Alice and Bob keys. Each person who wants to participate in the
blockchain should generate their own keys. This page explains several options for generating keys,
and each participant only needs to choose one such option. Regardless of which option you choose, be
sure to record all of the output from this section as you will need it later.

### Option 1: use Subkey

Subkey is a tool that generates keys specifically designed to be used with Substrate.

Begin by compiling and installing the utility (<a target="_blank" href="/v3/tools/subkey">instructions and more info here</a>). This may take up to 15 minutes or so.

We will need to generate at least **2** keys from each type. Every node will need to have its own
keys.

Generate a mnemonic and see the `sr25519` key and address associated with it. This key will be used
by Aura for block production.

```bash
# subkey command
subkey generate --scheme sr25519
```

```bash
# subkey output
Secret phrase `infant salmon buzz patrol maple subject turtle cute legend song vital leisure` is account:
  Secret seed:      0xa2b0200f9666b743402289ca4f7e79c9a4a52ce129365578521b0b75396bd242
  Public key (hex): 0x0a11c9bcc81f8bd314e80bc51cbfacf30eaeb57e863196a79cccdc8bf4750d21
  Account ID:       0x0a11c9bcc81f8bd314e80bc51cbfacf30eaeb57e863196a79cccdc8bf4750d21
  SS58 Address:     5CHucvTwrPg8L2tjneVoemApqXcUaEdUDsCEPyE7aDwrtR8D
```

Now see the `ed25519` key and address associated with the same mnemonic. This key will be used by
GRANDPA for block finalization.

```bash
# subkey command
subkey inspect --scheme ed25519 "infant salmon buzz patrol maple subject turtle cute legend song vital leisure"
```

```bash
# subkey output
Secret phrase `infant salmon buzz patrol maple subject turtle cute legend song vital leisure` is account:
  Secret seed:      0xa2b0200f9666b743402289ca4f7e79c9a4a52ce129365578521b0b75396bd242
  Public key (hex): 0x1a0e2bf1e0195a1f5396c5fd209a620a48fe90f6f336d89c89405a0183a857a3
  Account ID:       0x1a0e2bf1e0195a1f5396c5fd209a620a48fe90f6f336d89c89405a0183a857a3
  SS58 Address:     5CesK3uTmn4NGfD3oyGBd1jrp4EfRyYdtqL3ERe9SXv8jUHb
```

### Option 2: use Polkadot-JS Apps

The same UI that we used to see blocks being produced can also be used to generate keys. This option
is convenient if you do not want to install Subkey. It can be used for production keys, but the
system should not be connected to the internet when generating such keys.

<Message
  type={`red`}
  title={`Warning`}
  text="A system that generates production keys should not be connected to the internet regardless of what
    method you choose. It is mentioned here specifically because having an internet connection is 
    generally desired when using a webapp like Polkadot JS Apps.
    "
/>{' '}

On the "Accounts" tab, click "Add account". You do not need to provide a name, although you may if
you would like to save this account for submitting transaction in addition to validating.

Generate an `sr25519` key which will be used by Aura for block production. Take careful note of the
mnemonic phrase, and the SS58 address which can be copied by clicking on the identicon in the top
left.

Then generate an `ed25519` key which will be used by GRANDPA for block finalization. Again, note
the mnemonic phrase and ss58 address.

### Option 3: Use Pre-Generated Keys

If you just want to get on with the tutorial, you may use one of the pre-generated keypairs below.
But realize that these keys should absolutely not be used in production, and are provided only for
learning purposes.

##### Pair 1

| Key           | Value                                                                  |
| ------------- | ---------------------------------------------------------------------- |
| Secret phrase | `clip organ olive upper oak void inject side suit toilet stick narrow` |
| Secret seed   | `0x4bd2b2c1dad3dbe3fa37dc6ad5a4e32ddd8ad84b938179ac905b0622880e86e7`   |
| **SR25519**   |                                                                        |
| Public key    | `0x9effc1668ca381c242885516ec9fa2b19c67b6684c02a8a3237b6862e5c8cd7e`   |
| SS58 Address  | `5FfBQ3kwXrbdyoqLPvcXRp7ikWydXawpNs2Ceu3WwFdhZ8W4`                     |
| **ED25519**   |                                                                        |
| Public key    | `0xb48004c6e1625282313b07d1c9950935e86894a2e4f21fb1ffee9854d180c781`   |
| SS58 Address  | `5G9NWJ5P9uk7am24yCKeLZJqXWW6hjuMyRJDmw4ofqxG8Js2`                     |

##### Pair 2

| Key           | Value                                                                        |
| ------------- | ---------------------------------------------------------------------------- |
| Secret phrase | `paper next author index wedding frost voice mention fetch waste march tilt` |
| Secret seed   | `0x4846fedafeed0cf307da3e2b5dfa61415009b239119242006fc8c0972dde64b0`         |
| **SR25519**   |                                                                              |
| Public key    | `0x74cca68a32156615a5923c67024db70da5e7ed36e70c8cd5bcf3556df152bb6d`         |
| SS58 Address  | `5EhrCtDaQRYjVbLi7BafbGpFqcMhjZJdu8eW8gy6VRXh6HDp`                           |
| **ED25519**   |                                                                              |
| Public key    | `0x0fe9065f6450c5501df3efa6b13958949cb4b81a2147d68c14ad25366be1ccb4`         |
| SS58 Address  | `5CRZoFgJs4zLzCCAGoCUUs2MRmuD5BKAh17pWtb62LMoCi9h`                           |

In the next section, we will create a custom chain specification using your key pairs and
share your custom chain specification with your network validators

## Create a custom chain spec

Now that each participant has their own keys generated, you're ready to create a custom chain
specification. We will use this custom chain spec instead of the built-in `local` spec that we used
previously.

In this example we will create a two-node network, but the process generalizes to more nodes in a
straight-forward manner.

### Create a chain specification

Last time around, we used `--chain local` which is a predefined "chain spec" that has Alice and Bob
specified as validators along with many other useful defaults.

Rather than writing our chain spec completely from scratch, we'll just make a few modifications to
the one we used before. To start, we need to export the chain spec to a file named
`customSpec.json`.

<Message
  type={`green`}
  title={`Tip`}
  text={`Further details about all of these commands are available by running: \`node-template --help\`.`}
/>

```bash
# Export the local chain spec to json
./target/release/node-template build-spec --disable-default-bootnode --chain local > customSpec.json
```

### Modify Aura authority nodes

The file we just created contains several fields, and you can learn a lot by exploring them. By far
the largest field is a single binary blob that is the Wasm binary of our runtime. It is part of what
you built earlier when you ran the `cargo build --release` command.

The portion of the file we're interested in is the Aura authorities used for creating blocks,
indicated by **"aura"** field below, and GRANDPA authorities used for finalizing blocks, indicated
by the **"grandpa"** field. That section looks like this using the [two provided demo keys](#option-3-use-pre-generated-keys):

```json
{
  //-- snip --
  "genesis": {
    "runtime": {
      "frameSystem": {
        //-- snip --
      },
      "palletAura": {
        "authorities": [
          "5FfBQ3kwXrbdyoqLPvcXRp7ikWydXawpNs2Ceu3WwFdhZ8W4",
          "5EhrCtDaQRYjVbLi7BafbGpFqcMhjZJdu8eW8gy6VRXh6HDp"
        ]
      },
      "palletGrandpa": {
        "authorities": [
          ["5G9NWJ5P9uk7am24yCKeLZJqXWW6hjuMyRJDmw4ofqxG8Js2", 1],
          ["5CRZoFgJs4zLzCCAGoCUUs2MRmuD5BKAh17pWtb62LMoCi9h", 1]
        ]
      }
      //-- snip --
    }
  }
}
```

<br />
<Message
  type={`yellow`}
  title={`Information`}
  text="These instructions are written for the node template. They will work with other Substrate-based 
    nodes with few modifications. In Substrate, nodes that include the session pallet should leave 
    the Aura and Grandpa configs empty and instead insert this information in the session config. All 
    of this is demonstrated in the chain spec you export from your desired node. 
    "
/>

The format for the Grandpa data is more complex because the Grandpa protocol supports weighted
votes. In this case we have given each validator a weight of **1**.

All we need to do is change the authority addresses listed (currently Alice and Bob) to our own
addresses that we generated in the previous step. The **sr25519** addresses go in the **aura**
section, and the **ed25519** addresses in the **grandpa** section. You may add as many validators as
you like. For additional context, read about
[keys in Substrate](/v3/advanced/cryptography#public-key-cryptography).

<Message
  type={`red`}
  title={`Warning`}
  text="Validators should not share the same keys, even for learning purposes. If two validators have the 
    same keys, they will produce conflicting blocks. 
    A single person should create the chain spec and share the resulting 
    `customSpecRaw.json` file with their fellow validators. 
    "
/>{' '}

### Convert and share the raw chain spec

Once the chain spec is prepared, convert it to a "raw" chain spec. The raw chain spec contains all
the same information, but it contains the encoded storage keys that the node will use to reference
the data in its local storage. Distributing a raw spec ensures that each node will store the data at
the proper storage keys.

```bash
./target/release/node-template build-spec --chain=customSpec.json --raw --disable-default-bootnode > customSpecRaw.json
```

Finally share the `customSpecRaw.json` with your all the other validators in the network.

<Message
  type={`yellow`}
  title={`Information`}
  text="Because Rust -> Wasm optimized builds aren't reproducible, each person will get a slightly
    different Wasm blob which will break consensus if each participant generates the file themselves.
    For the curious, learn more about this issue in 
    [this blog post](https://dev.to/gnunicorn/hunting-down-a-non-determinism-bug-in-our-rust-wasm-build-4fk1).
    "
/>

Next steps:

- Launch the chain with the bootnode
- Add keys to keystore
- Add other participants to your network

## Launch your private network

With your custom chain spec created and distributed to all participants, you're ready to launch your
own custom chain. In this section it is no longer required to use a single physical machine or a
single binary.

### First participant starts a bootnode

You've completed all the necessary prep work and you're now ready to launch your chain. This process
is very similar to when you launched a chain earlier, as Alice and Bob. It's important to start with
a clean base path, so if you plan to use the same path that you've used previously, please delete
all contents from that directory.

The first participant can launch her node with:

```bash
# purge chain (only required for new/modified dev chain spec)
./target/release/node-template purge-chain --base-path /tmp/node01 --chain local -y
```

```bash
# start node01
./target/release/node-template \
  --base-path /tmp/node01 \
  --chain ./customSpecRaw.json \
  --port 30333 \
  --ws-port 9945 \
  --rpc-port 9933 \
  --telemetry-url 'wss://telemetry.polkadot.io/submit/ 0' \
  --validator \
  --rpc-methods Unsafe \
  --name MyNode01
```

Here are some differences from when we launched as Alice.

- I've omitted the `--alice` flag. Instead we will insert our own custom keys into the keystore
  through the RPC shortly.
- The `--chain` flag has changed to use our custom chain spec.
- I've added the optional `--name` flag. You may use it to give your node a human-readable name in
  the telemetry UI.
- The optional `--rpc-methods Unsafe` flag has been added. As the name indicates, this flag is not
  safe to use in a production setting, but it allows this tutorial to stay focused on the topic at
  hand.

You should see the console outputs something as follows:

```bash
2021-03-10 18:32:15  Substrate Node
2021-03-10 18:32:15  ✌️  version 3.0.0-1c5b984-x86_64-linux-gnu
2021-03-10 18:32:15  ❤️  by Substrate DevHub <https://github.com/substrate-developer-hub>, 2017-2021
2021-03-10 18:32:15  📋 Chain specification: Local Testnet
2021-03-10 18:32:15  🏷 Node name: MyNode01
2021-03-10 18:32:15  👤 Role: AUTHORITY
2021-03-10 18:32:15  💾 Database: RocksDb at /tmp/node01/chains/local_testnet/db
2021-03-10 18:32:15  ⛓  Native runtime: node-template-100 (node-template-1.tx1.au1)
2021-03-10 18:32:16  🔨 Initializing Genesis block/state (state: 0xea47…9ba8, header-hash: 0x9d07…7cce)
2021-03-10 18:32:16  👴 Loading GRANDPA authority set from genesis on what appears to be first startup.
2021-03-10 18:32:16  ⏱  Loaded block-time = 6000 milliseconds from genesis on first-launch
2021-03-10 18:32:16  Using default protocol ID "sup" because none is configured in the chain specs
2021-03-10 18:32:16  🏷 Local node identity is: 12D3KooWJvVUoAa7R8gjCSQ45x69Ahh3HcdVSH1dvpcA52vKawHL
2021-03-10 18:32:16  📦 Highest known block at #0
2021-03-10 18:32:16  〽️ Prometheus server started at 127.0.0.1:9615
2021-03-10 18:32:16  Listening for new connections on 127.0.0.1:9944.
2021-03-10 18:32:21  💤 Idle (0 peers), best: #0 (0x9d07…7cce), finalized #0 (0x9d07…7cce), ⬇ 0 ⬆ 0
2021-03-10 18:32:26  💤 Idle (0 peers), best: #0 (0x9d07…7cce), finalized #0 (0x9d07…7cce), ⬇ 0 ⬆ 0
```

Here you must take note of:

- the **node identity**: `12D3KooWJvVUoAa7R8gjCSQ45x69Ahh3HcdVSH1dvpcA52vKawHL`
- the **IP address** `127.0.0.1`
- the p2p port `--port = 30333`

These values are for this specific
example, but for your node, they will be different and **required for other nodes to directly connect to it**
(without a bootnode in the chain spec, as we removed in the flags before).

### Add keys to keystore

Once your node is running, you will again notice that no blocks are being produced. At this point,
you need to add your keys into the keystore. Remember you will need to complete these steps for each
node in your network. You will add two types of keys for each node: Aura and GRANDPA keys. Aura keys
are necessary for
[block _production_](/v3/getting-started/glossary#author);
GRANDPA keys are necessary for
[block _finalization_](/v3/getting-started/glossary#finality).

#### Option 1: use the Polkadot-JS Apps UI

You can use the Apps UI to insert your keys into the keystore. Navigate to "Developer --> RPC Call". Choose
"author" and "insertKey". The fields can be filled like this:

![Inserting a Aura key using Apps](../img/tutorials/05-private-network/private-network-apps-insert-key-aura.png)

```
keytype: aura
suri: <your mnemonic phrase> (eg. clip organ olive upper oak void inject side suit toilet stick narrow)
publicKey: <your raw sr25519 key> (eg. 0x9effc1668ca381c242885516ec9fa2b19c67b6684c02a8a3237b6862e5c8cd7e)
```

<br />
<Message
  type={`gray`}
  title={`Note`}
  text="If you generated your keys with the Apps UI you will not know your raw public key. In this case
      you may use your SS58 address (`5FfBQ3kwXrbdyoqLPvcXRp7ikWydXawpNs2Ceu3WwFdhZ8W4`) instead.
      "
/>

You've now successfully inserted your
[**Aura**](/v3/getting-started/glossary#aura-aka-authority-round)
key. You can repeat those steps to insert your
[**GRANDPA**](/v3/getting-started/glossary#grandpa) key (the
**ed25519** key)

![Inserting a GRANDPA key using Apps](../img/tutorials/05-private-network/private-network-apps-insert-key-gran.png)

```
keytype: gran
suri: <your mnemonic phrase> (eg. clip organ olive upper oak void inject side suit toilet stick narrow)
publicKey: <your raw ed25519 key> (eg. 0xb48004c6e1625282313b07d1c9950935e86894a2e4f21fb1ffee9854d180c781)
```

<br />
<Message
  type={`gray`}
  title={`Note`}
  text="If you are following these steps for the _second_ node in the network, you must connect the UI to 
      the second node before inserting the keys.
      "
/>

#### Option 2: use `curl`

You can also insert a key into the keystore by using [`curl`](https://curl.haxx.se/) from the
command line. This approach may be preferable in a production setting, where you may be using a
cloud-based virtual private server.

Because security is of the utmost concern in a production environment, it is important to take every
precaution possible. In this case, that means taking care that you do not leave any traces of your
keys behind, such as in your terminal's history. Create a file that you will use to define the body
for your `curl` request:

```json
{
  "jsonrpc": "2.0",
  "id": 1,
  "method": "author_insertKey",
  "params": ["<aura/gran>", "<mnemonic phrase>", "<public key>"]
}
```

```bash
# Submit a new key via RPC, connect to where your `rpc-port` is listening
curl http://localhost:9933 -H "Content-Type:application/json;charset=utf-8" -d "@/path/to/file"
```

If you enter the command and parameters correctly, the node will return a JSON response as follows.

```json
{ "jsonrpc": "2.0", "result": null, "id": 1 }
```

Make sure you delete the file that contains the keys when you are done.

#### Option 3: use the `key` command

Alternatively, you can insert a key saved to a local file using the `key` command:

```bash
# Insert the key from /path/to/key/file into the keystore
./target/release/node-template key insert --base-path /tmp/node01 --chain local --key-type <aura/gran> --suri /path/to/key/file
```

### Verify keys in the keystore (Optional)

Optionally, If you would like to check that your keys are now loaded, you can view the keystore files that should
now exists for your `node01`. These are found in the following (default example) location:

```bash
# The path stems from `--base-path` and ID from `chain_spec.rs` ID field.
# Keys are then in `chains/<chain ID>/keystore :
ls /tmp/node01/chains/local_testnet/keystore
```

```bash
## list of keystore files:
617572619effc1668ca381c242885516ec9fa2b19c67b6684c02a8a3237b6862e5c8cd7e
6772616eb48004c6e1625282313b07d1c9950935e86894a2e4f21fb1ffee9854d180c781

# read a keystore file (our demo seed 1 was used)
cat /tmp/node01/chains/local_testnet/keystore/617572619effc1668ca381c242885516ec9fa2b19c67b6684c02a8a3237b6862e5c8cd7e
"clip organ olive upper oak void inject side suit toilet stick narrow"
```

Notice there are two keystores, as expected as we added two keys to our node.
This example used [pair 1](#option-3-use-pre-generated-keys) from the well known seeds here, and
will use `pair 2` for our next node, where your keys may differ.

### Subsequent participants join

Subsequent validators can now join the network. This can be done by specifying the `--bootnodes`
parameter as Bob did previously.

```bash
# purge chain (only required for new/modified dev chain spec)
./target/release/node-template purge-chain --base-path /tmp/node02 --chain local -y

# start node02
./target/release/node-template \
  --base-path /tmp/node02 \
  --chain ./customSpecRaw.json \
  --port 30334 \
  --ws-port 9946 \
  --rpc-port 9934 \
  --telemetry-url 'wss://telemetry.polkadot.io/submit/ 0' \
  --validator \
  --rpc-methods Unsafe \
  --name MyNode02 \
  --bootnodes /ip4/127.0.0.1/tcp/30333/p2p/12D3KooWAvdwXzjmRpkHpz8PzUTaX1o23SdpgAWVyTGMSQ68QXK6
  # you MUST fill the correct info in the line above:
  # --bootnodes /ip4/<IP Address>/tcp/<p2p Port>/p2p/<Peer ID>
```

<br />
<Message
  type={`red`}
  title={`Warning`}
  text="Make sure you set the correct node ID! If you didn\'t set the correct node ID gathered from another running with the same chain spec,
    but only the right IP - you will get errors of the form: 
    `💔 The bootnode you want to connect to at ... provided a different peer ID than the one you expect: ...`
    "
/>

As before, we specify another `base-path`, give it another `name`, and also specify this node as a
`validator`.

**Block production**: Now you _must_ also set the authoring keys in this node, just as we did
[for the first node](#add-keys-to-keystore). Note that you will need to communicate with your node
on the correct `ws-port` (so setting the app UI and submitting `curl` to the right port is critical)
A node will not be able to produce blocks if it has not added its Aura key!
**Block finalization**: This can _only_ happen if more than two-thirds of the validators have added their
GRANDPA keys to their keystores. Since this network was configured with two validators (in the
chain spec), block finalization can occur after the second node has added its keys (i.e. 50% < 66% < 100%).

**Reminder:** All validators must be using _identical chain specifications_ in order to peer. You should
see the same genesis block and state root hashes.

Once the second node is running _and_ it has an authoring key, you should see both nodes reporting block authoring:

```bash
2021-03-18 16:43:10  Substrate Node
2021-03-18 16:43:10  ✌️  version 3.0.0-c528fd2-x86_64-linux-gnu
2021-03-18 16:43:10  ❤️  by Substrate DevHub <https://github.com/substrate-developer-hub>, 2017-2021
2021-03-18 16:43:10  📋 Chain specification: Local Testnet
2021-03-18 16:43:10  🏷 Node name: MyNode02
2021-03-18 16:43:10  👤 Role: AUTHORITY
2021-03-18 16:43:10  💾 Database: RocksDb at /tmp/node02/chains/local_testnet/db
2021-03-18 16:43:10  ⛓  Native runtime: node-template-100 (node-template-1.tx1.au1)
2021-03-18 16:43:10  Using default protocol ID "sup" because none is configured in the chain specs
2021-03-18 16:43:10  🏷 Local node identity is: 12D3KooWDfpXmPtsvLCFTh4mKybYi6MvDMDiUrnRDcGZiSTR2GHp
2021-03-18 16:43:10  📦 Highest known block at #1
2021-03-18 16:43:10  Listening for new connections on 127.0.0.1:9946.
2021-03-18 16:43:11  🔍 Discovered new external address for our node: /ip4/127.0.0.1/tcp/30334/p2p/12D3KooWDfpXmPtsvLCFTh4mKybYi6MvDMDiUrnRDcGZiSTR2GHp
2021-03-18 16:43:12  🙌 Starting consensus session on top of parent 0x700fda8b9c7574553eccc8acc72e2dec59e40711e743223d67c3e5b57e1f76ef
2021-03-18 16:43:12  ♻️  Reorg on #1,0x700f…76ef to #2,0xe111…c084, common ancestor #0,0x2776…8ba7
2021-03-18 16:43:12  ✨ Imported #2 (0xe111…c084)
2021-03-18 16:43:12  Timeout fired waiting for transaction pool at block #1. Proceeding with production.
2021-03-18 16:43:12  🎁 Prepared block for proposing at 2 [hash: 0xc590a846ff17871ffdcdd670914321f667cd6ad0b898bfb6d25f7dd68fff478b; parent_hash: 0x700f…76ef; extrinsics (1): [0x34d6…ed56]]
2021-03-18 16:43:12  🔖 Pre-sealed block for proposal at 2. Hash now 0x38b29b343d8f6ef56286a2f3aad20c82ae75d5bb0698569dd06fca654dae6fa6, previously 0xc590a846ff17871ffdcdd670914321f667cd6ad0b898bfb6d25f7dd68fff478b.
2021-03-18 16:43:12  ✨ Imported #2 (0x38b2…6fa6)
2021-03-18 16:43:15  💤 Idle (1 peers), best: #2 (0xe111…c084), finalized #0 (0x2776…8ba7), ⬇ 1.4kiB/s ⬆ 1.4kiB/s
```

The final lines shows that your node has peered with another (**`1 peers`**), and they have produced
a block (**`best: #2 (0xe111…c084)`**)!

But do notice that even after you add the GRANDPA key for the second node no block finalization has
happened (**`finalized #0 (0x2776…8ba7)`**).
**Substrate nodes require a restart after inserting a GRANDPA key.**
Kill your nodes and restart them with the same commands you used previously.
Now blocks should be finalized!

```bash
...
2021-03-18 16:47:47  💤 Idle (1 peers), best: #46 (0xfaf1…02f8), finalized #44 (0x9b08…09ea), ⬇ 1.3kiB/s ⬆ 1.3kiB/s
2021-03-18 16:47:48  ✨ Imported #47 (0x7375…aa51)
2021-03-18 16:47:52  💤 Idle (1 peers), best: #47 (0x7375…aa51), finalized #45 (0x7c13…7575), ⬇ 0.8kiB/s ⬆ 0.6kiB/s
2021-03-18 16:47:54  🙌 Starting consensus session on top of parent 0x73757e1773e6d86a9ef4a3ec9c3a55eef04345705c0a51f04445af657184aa51
2021-03-18 16:47:54  🎁 Prepared block for proposing at 48 [hash: 0xd9ef428ccd38426a47a9eca181b508630e327b35ef4c468103ce59fa861e60f6; parent_hash: 0x7375…aa51; extrinsics (1): [0x16f0…dbe6]]
2021-03-18 16:47:54  🔖 Pre-sealed block for proposal at 48. Hash now 0x23a93d8e6bbbcf9f36e61264cc3a48a426a7f1112ff48df76f0d55b52c181156, previously 0xd9ef428ccd38426a47a9eca181b508630e327b35ef4c468103ce59fa861e60f6.
```

Congratulations! You've started your own private blockchain!

In this tutorial you've learned to generate your own keypairs, create a custom chain spec that uses
those keypairs, and start a private network based on your custom chain spec.

<Message
  type={`green`}
  title={`Tip`}
  text={`If you would like examples of correct JSON keystore \`curl\` files with a known working chain spec, see [this
  solution](https://github.com/substrate-developer-hub/substrate-node-template/tree/tutorials/solutions/private-chain-v3). 
  You can use the extra files with your already cloned and compiled node, no need to start from scratch.`}
/>

## Next steps

That big Wasm blob we encountered in the chain spec was necessary to enable forkless upgrades.
Learn more about how the [executor](/v3/advanced/executor) uses on-chain Wasm. Alternatively, continue
learning by completing the [permissioned network tutorial](/tutorials/v3/permissioned-network).