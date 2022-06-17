![](https://img.shields.io/badge/status-wip-orange.svg?style=flat-square)

# Store

-----

**Abstract**

Store modules define the operations and a state index for a specific store.

# Table of Contents

- [Introduction](#introduction)
- [Operations](#operations)
  - [Example Manifest](#example-manifest)
- [Manifest Address](#manifest-address)
  - [String Format](#address-string-format)
  - [Binary Format](#address-binary-format)

## Introduction

Stores are used to derive a database state from the database replica. They provide an api for read/write to the database. Different stores can be used to represent very different structures.

## Operations

Operations are used to update database states. They are added to entry payloads and read by the store to derive state. In this context an operation is an object with an `op` field that has a string value. The object can have any other fields with other values. The values must be able to be encoded in CBOR since it will be placed inside the entry.

###### see cbor [encoding specification](https://www.rfc-editor.org/rfc/rfc8949.html#name-specification-of-the-cbor-e)

```
{
  op: type 3
}
```


#### Example Kvstore Put Operation:

```
  {
    op: 'PUT',
    key: 'example',
    value: 123
  }
```

## Index

The Index defines the structure of the database state. Indexes read the entry log from the replica to derive the index state. It reads the entry log

TODO: example definition for reducing an index from operations
