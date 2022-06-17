![](https://img.shields.io/badge/status-wip-orange.svg?style=flat-square)

# Replicator

-----

**Abstract**

A replicator finds and syncs the database replica with peers. They can read the local replica and advertise new updates to peers. They can push advertised entries and identities to be added to the local replica.

# Table of Contents

- [Introduction](#introduction)
- [Implementations](#implementations)

## Introduction

OrbitDB replicator modules handle syncing the database replica between peers. The database replica includes the manifest, entries, and identities. However replicators are mainly concerned with the latter two as the manifest is needed before replication, and does not get updated.

## Implementations

- [pubsub-heads-exchange](./pubsub-heads-exchange)
  - [version 1](./pubsub-heads-exchange/1.md)
