# go-ds-crdt

> A distributed [go-datastore](https://github.com/ipfs/go-datastore)
> implementation using Merkle-CRDTs.

`go-ds-crdt` is a key-value store implementation using Merkle CRDTs, as
described in
[the paper by Héctor Sanjuán, Samuli Pöyhtäri and Pedro Teixeira](https://hector.link/presentations/merkle-crdts/merkle-crdts.pdf).
It satisfies the
[`Datastore`](https://godoc.org/github.com/ipfs/go-datastore#Datastore)
and [`Batching`](https://godoc.org/github.com/ipfs/go-datastore#Batching)
interfaces from `go-datastore`.

Internally it uses a delta-CRDT Add-Wins Observed-Removed set. The current
value for a key is the one with highest priority. Priorities are defined as
the height of the Merkle-CRDT node in which the key was introduced.

## Usage

`go-ds-crdt` needs:
  * A user-provided, thread-safe,
    [`go-datastore`](https://github.com/ipfs/go-datastore) implementation to
    be used as permanent storage. We recommend using the
    [Badger implementation](https://godoc.org/github.com/ipfs/go-ds-badger).
  * A user-defined `Broadcaster` component to broadcast and receive updates
    from a set of replicas. If your application uses
    [libp2p](https://libp2p.io), you can use
    [libp2p PubSub](https://godoc.org/github.com/libp2p/go-libp2p-pubsub) and
    the provided
    [`PubsubBroadcaster`](https://godoc.org/github.com/ipfs/go-ds-crdt#PubSubBroadcaster).
  * A user-defined `DAGSyncer` component to publish and retrieve Merkle DAGs
    to the network. For example, you can use
    [IPFS-Lite](https://github.com/hsanjuan/ipfs-lite) which casually
    satisfies this interface.

The permanent storage layout is optimized for KV stores with fast indexes and
key-prefix support.

See https://godoc.org/github.com/ipfs/go-ds-crdt for more information.

## How to use

### 1. Navigate to the following folder and update the `config.yaml` file

``` sh
$ cd examples/globaldb
```

Update the `config.yaml` file in the examples/globaldb folder, preferably with the multiaddr of your IPFS node.

### 2. Run globaldb.go along with the argument `daemon`

``` sh
$ go run globaldb.go daemon
Bootstrapping...
Connecting to the bootstrap node specified in config.yaml:  /ip4/A.B.C.D/tcp/4001/p2p/QmABC
Peer ID: Qm123
Listen address: /ip4/0.0.0.0/tcp/33123
Topic: globaldb-example
Data Folder: /Users/$USER/globaldb-example

Ready!

Commands:

> list               -> list items in the store
> get <key>          -> get value for a key
> put <key> <value>  -> store value on a key
> exit               -> quit


Running in daemon mode
>
```

### 3. Add a Key-Value entry using put

```sh
> put "IPFS" "Permanent web"
Added: [/"IPFS"] -> "Permanent web"
Apr  9 18:50:55 - 1 connected peers
```

The last line displays the fellow IPFS nodes connected, as you are running in daemon mode.

### 4. Read a Key-Value entry using get

```sh
> get "IPFS"
[/"IPFS"] -> "Permanent web"
Apr  9 18:52:27 - 2 connected peers
```

### 5. List all the Key-Value entries stored locally

```sh
> list
[/"IPFS"] -> "Permanent web"
[/hey] -> you
[/heyy] -> baby
[/utthista] -> bhaaratha
Apr  9 18:53:28 - 3 connected peers
```

You can see all the Key-Value entries stored locally.

### 6. View all the connections in action

```sh
$ lsof -i :33123
COMMAND    PID      USER   FD   TYPE             DEVICE SIZE/OFF NODE NAME
globaldb 21989 $USERNAME   21u  IPv4 0xbc8c7df5d7c21d1b      0t0  TCP *:33123 (LISTEN)
globaldb 21989 $USERNAME   22u  IPv4 0xbc8c7df5bda1802b      0t0  TCP 192.168.1.7:33123->mars.i.ipfs.io:newoak (ESTABLISHED)
globaldb 21989 $USERNAME   26u  IPv4 0xbc8c7df5d83589b3      0t0  TCP 192.168.1.7:33123->pluto.i.ipfs.io:newoak (ESTABLISHED)
globaldb 21989 $USERNAME   27u  IPv4 0xbc8c7df5d7d0e33b      0t0  TCP 192.168.1.7:33123->207.148.19.196.vultr.com:commtact-http (ESTABLISHED)
globaldb 21989 $USERNAME   28u  IPv4 0xbc8c7df5d7d0bd1b      0t0  TCP 192.168.1.7:33123->earth.i.ipfs.io:newoak (ESTABLISHED)
globaldb 21989 $USERNAME   30u  IPv4 0xbc8c7df5d7dba6a3      0t0  TCP 192.168.1.7:33123->venus.i.ipfs.io:newoak (ESTABLISHED)
globaldb 21989 $USERNAME   31u  IPv4 0xbc8c7df5c0f64d1b      0t0  TCP 192.168.1.7:33123->saturn.i.ipfs.io:newoak (ESTABLISHED)
globaldb 21989 $USERNAME   34u  IPv4 0xbc8c7df5bd36d02b      0t0  TCP 192.168.1.7:33123->ns361433.ip-91-121-168.eu:54001 (ESTABLISHED)
globaldb 21989 $USERNAME   35u  IPv4 0xbc8c7df5d83be02b      0t0  TCP 192.168.1.7:33123->14.152.102.108:22292 (ESTABLISHED)
globaldb 21989 $USERNAME   46u  IPv4 0xbc8c7df5cb8816a3      0t0  TCP 192.168.1.7:33123->ec2-18-205-194-29.compute-1.amazonaws.com:newoak (ESTABLISHED)
globaldb 21989 $USERNAME   58u  IPv4 0xbc8c7df5c8e4b33b      0t0  TCP 192.168.1.7:33123->177.0.222.35.bc.googleusercontent.com:30657 (ESTABLISHED)
globaldb 21989 $USERNAME   84u  IPv4 0xbc8c7df5cc7779b3      0t0  TCP 192.168.1.7:33123->212.129.234.29:newoak (ESTABLISHED)
```

### 7. Stop the daemon

Daemon mode currently has an issue with signal interruption. I hope to fix it soon!

Until then, daemon can be stopped using the following command:

``` sh
$ pkill -f "globaldb"
```

## Captain

This project is captained by @hsanjuan.

## License

This library is dual-licensed under Apache 2.0 and MIT terms.

Copyright 2019. Protocol Labs, Inc.
