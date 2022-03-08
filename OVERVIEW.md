# ![](https://img.shields.io/badge/status-wip-orange.svg?style=flat-square)

# Protocol Overview

-----

**Abstract**

OrbitDB is a **serverless, distributed, peer-to-peer database**. OrbitDB stores use [CRDTs](https://en.wikipedia.org/wiki/Conflict-free_replicated_data_type) for [strong eventual consistency](https://en.wikipedia.org/wiki/Eventual_consistency#Strong_eventual_consistency) and conflict-free database merges; and [IPLD](https://ipld.io), [IPFS](https://ipfs.io) and [Libp2p](https://libp2p.io) to immutably address and directly communicate updates. These properties make well-suited for decentralized and [local-first](https://www.inkandswitch.com/local-first/) applications.

# Table of Contents

- [Introduction](#introduction)
- [Replica](#replica)
- [Processes](#processes)
- [Unified](#unified)

## Introduction

The OrbitDB protocol defines ways to create and participate in a shared state with other peers. Each peer maintains its own copy of the database.
Instead of reading from a remote database, the peer's local database replica is used as the source of truth.

A participant can read and write to the database while offline. Updates they made while disconnected from peers can be communicated later in a conflict-free way. Peers with the same set of updates will have the same database state.

## Replica

The replica is made up of data which is later processed into the database state. It is also the part of the database that gets passed between peers. There are three data formats that make up the database replica. They are called the manifest, entry, and identity.

The [manifest](./MANIFEST.md) is a setup document. It contains configuration for the database. Nodes that setup their replica with the manifest configuration will satisfy strong eventual consistency; they require no additional agreement or coordination.

The [entry](./entry) is a CRDT with a payload. Each entry is a DAG node with causal links to previous entries. They make up the immutable log of updates to the database store. These make up the entry log aka operation log.

The [identity](./identity) contains a public key for signing entries. Each entry has an identity associated with it for verification of the entry signature and access control.

<img src="./.assets/replica_diagram.png" width="750" />

A database replica is made up of a manifest, and sets of entry and identity.

## Processes

The replica is processed by a few different components: store, access, and replicator.

The [store](./store) provides the interface to read and write to the database.

The [access](./access) provides access control/write protection to the database; enforced by peers on their own replica.

A [replicator](./replicator) handles syncing the replica across participating devices.

## Unified

Together the replica and processes make up the database. The first part to exist of a replica is the manifest.

Only entries with valid signatures, signed by identities allowed by the access module, can be added to the replica. Entries created locally using the store methods and entries replicated from peers must both pass this threshold before being added.

In the case of mutable access control lists, entries may be removed from the replica if the identity that created them is revoked access.

Anytime the replica is changed, by adding or removing entries and identities, the store index will need to be recalculated.
