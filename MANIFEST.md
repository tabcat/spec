![](https://img.shields.io/badge/status-wip-orange.svg?style=flat-square)

# OrbitDB Manifest

-----

**Abstract**

OrbitDB databases all have an immutable setup document called a manifest. The manifest contains information needed to run a database. This information specifies modules/formats and their versions to use to when running the database.

# Table of Contents

- [Introduction](#introduction)
- [Manifest Format](#manifest-format)
  - [Example Manifest](#example-manifest)
- [Manifest Address](#manifest-address)
  - [String Format](#address-string-format)
  - [Binary Format](#address-binary-format)

## Introduction

The manifest must contain relevant information to running the database in agreement with other participants. Some of the information that is part of the manifest includes the databases name and type as well as modules and formats to use.

Many of the fields specify a module type, sometimes with associated configuration. This type, which is just a string, is used by the implementation to load the correct software and version needed for running that piece of the database.

All information needed for OrbitDB to run the database must be included with the manifest, not just referenced. This rule makes it easy to know whether a database can be opened or not.

## Manifest Format

An OrbitDB manifest is a data structure, encoded using dag-cbor [[spec](https://github.com/ipld/ipld/blob/master/specs/codecs/dag-cbor/spec.md)], containing the following fields:

- **version** (uint64)
  - The version of the manifest
  - This document specifies version 1
  - Set to `1`


- **name** (utf8)
  - The name of the database
  - Chosen by the user
  - Does not need to be unique
  - No *default* value


- **type** (utf8)
  - The type of the database to be used. Read more about [database stores](./store)
  - A string key to find the correct store module
  - No *default* value


- **access** (map)
  - Contains the access type and configuration. Read more about [access modules](./access)
  - Must contain a `.type` property for the access modules type to be set
  - May contain other configuration for the access module
  - *default*: The default access module type is `'/orbitdb/access/ipfs/1.0.0'`


- **replicator** (map)
  - Contains the replicator types and configuration. Read more about [replicator modules](./replicator).
  - Must contain a `.types` property for the replicator modules type to be set
  - May contain other configuration for the replicator modules
  - *default*: The default replicator module types are `['/orbitdb/replicator/pubsub-heads-exchange/1.0.0']`


- **entry** (utf8)
  - The entry type to be used. Read more about [entry formats](./entry).
  - *default*: The default entry format type is `'/ipfs-log/entry/3.0.0'`


- **identity** (utf8)
  - The identity type to be used. Read more about [identity formats](./identity).
  - *default*: The default identity format type is `'/orbitdb/identity/2.0.0'`


- **meta** (any) [optional]
  - An optional user-space field for including any information they want to keep immutable and associated with the database.
  - *default*: If no meta is supplied, then no meta field exists in the manifest and is treated as `null`.


- **tag** (bytes) [optional]
  - A way to refer to the database without revealing the manifest.
  - Must be unique
  - *default*: If no tag is supplied, then no tag field exists in the manifest. In this case the database tag must be treated as the SHA2-256 digest of the dag-cbor encoded manifest.



###### see cbor [encoding specification](https://www.rfc-editor.org/rfc/rfc8949.html#name-specification-of-the-cbor-e) and [tag 42](https://github.com/ipld/cid-cbor/)

```
  {
    version: type 0
    name: type 3
    type: type 3
    access: {
      type: type 3
    }
    replicator: {
      types: type 5<type3>
    }
    entry: type 3
    identity: type 3
    meta?: any
    tag?: type 2
  }
```

#### Example Manifest:

```
  {
    version: 1,
    name: 'example',
    type: '/orbitdb/kvstore/1.0.0',
    access: {
      type: '/orbitdb/access/ipfs/1.0.0',
      write: [
        CID(bafyreicl6ujc6ncfktctxxroxognfn7d2fqavvrryoc2lv6m4i6hpbkfti)
      ]
    },
    replicator: {
      types: ['/orbitdb/replicator/pubsub-heads-exchange/1.0.0']
    },
    entry: '/ipfs-log/entry/3.0.0',
    identity: '/orbitdb/identity/2.0.0'
  }
```

## Manifest Address

  Also known as the database address, these are used as reference to the database and for potential database peers to fetch the manifest. They are made of the protocol prefix for OrbitDB and a manifest CID.

#### Address String Format

  `/orbitdb/<manifest CID>`

  This is the manifest address string format. The version and encoding for the CID are not specified. Here are some examples of valid addresses:

  - `/orbitdb/QmWDUfC4zcWJGgc9UHn1X3qQ5KZqBv4KCiCtjnpMmBT8JC`
  - `/orbitdb/zdpuArEpCPwqpgQVMYrFSDQKhHLq5L6Es8HQkhWfhrad5oYdK`
  - `/orbitdb/bafybeidpga2xwiiseib3tcsp33wodu3itunwj2su624yygdx6av3s4f5f4`

#### Address Binary Format

  `<orbitdb multicodec><manifest CID>`

  This is the manifest address binary format. The multicodec code for orbitdb is `0xdb`. This byte must be prefixed to the manifest CID for it to be valid. Here are some examples using the same CID from the string section:

  - `0xdb1220750710ddc61a99b8604fc77e4067b53c24d0488cd28fda4cb344ffee95a73191`
  - `0xdb01711220566d493416aee3a6585402934b8544468b5d040935cb1dcb43830057966a0a46`
  - `0xdb017012206f30357b21122203b98a4fdeece1d3689d1b64ea54f6b98c1877f02bb970bd2f`
