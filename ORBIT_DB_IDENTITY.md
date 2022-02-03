![](https://img.shields.io/badge/status-wip-orange.svg?style=flat-square)

# OrbitDB Identity

-----

**Abstract**

OrbitDB Identities are use for verification and access control of database entries. They often contain keys used to digitally sign data and serve as a composable way to distribute access to multiple devices without sharing any private data.

# Table of Contents

- [Introduction](#introduction)
- [Format](#format)

## Introduction

The OrbitDB Identity is used for verification and access control of database entries. It is made up of two secp256k1 public keys and an ECDSA signature

The first public key is used for access control, it digitally signs the second public key. This digital signature is the ECDSA signature included with the identity. If the first public key exists in the database's access control then that identity may make entries to the database.

The second public key is used to sign the database entries. If both the entry signature and the identity signature are valid and the identity exists in the access control, then the entry is valid and is added to the database operation log.

## Format

An OrbitDB Identity is a data structure, encoded using dag-cbor [[spec](https://github.com/ipld/ipld/blob/master/specs/codecs/dag-cbor/spec.md)] and containing the following fields:

- **root** (bytes)
  - The identity root public key.
  - It must be a compressed* secp256k1 public key.
  - Used to for access control.
  - Key used to sign key in `pub` field.


- **pub** (bytes)
  - The identity public key.
  - It must be a compressed* secp256k1 public key.
  - Used to sign database entries.
  - May be the same key used in the `root` field.


- **sig** (bytes)
  - The signature of `root` field public key signing `pub` field public key bytes.
  - It must be an ECDSA signature.
  - Verifies that the `pub` field public key belongs to the `root` field public key.


###### see cbor [major types](https://www.rfc-editor.org/rfc/rfc8949.html#section-3.1) and [tag 42](https://github.com/ipld/cid-cbor/)

```
  {
    auth: type 2
    sig: type 2
    data: type 2
  }
```

\* a compressed secp public key refers to point compression defined in section 2.3.3-2.3.4 of [Standards for Efficient
Cryptography 1 (SEC 1)](https://www.secg.org/sec1-v2.pdf)
