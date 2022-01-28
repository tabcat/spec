![](https://img.shields.io/badge/status-wip-orange.svg?style=flat-square)
# IPFS-Log Entry

-----

**Abstract**

[IPFS-Log](https://github.com/orbitdb/ipfs-log) is a core part of [OrbitDB](https://orbitdb.org), it is used by the database as an operation log from which state is reduced. This log is built from entries added to the log. Entries are immutable and reference previous entries by [content-address](https://docs.ipfs.io/concepts/content-addressing) also called CID; creating causal and traversable links.

Besides references to previous entries, entries include other information:

 - version
 - database tag
 - logical clock
 - payload
 - creator

# Table of Contents

- [Introduction](#introduction)
- [Entry-Format](#entry-format)
- [Signature Creation and Verification](#signature-creation-and-verification)
- [Logical Clock](#logical-clock)
- [Reference Fields](#reference-fields)

## Introduction

IPFS-Log entries are used to append updates in an immutable and conflict free way. Peers share them with each other and add them to their local replica.
Before new entries can be added to the replica, their digital signature and writer identity need to be verified.

It's important to make sure the entry format is easy to transmit and verify. To do this, the spec takes advantage of binary types in CBOR to save space; and structures the data so verifying the digital signature is straightforward.

## Entry Format

An IPFS-Log Entry is a data structure, encoded using dag-cbor [[spec](https://github.com/ipld/ipld/blob/master/specs/codecs/dag-cbor/spec.md)] and containing the following fields:

- 0. **from** (CID)
  - The creator of the entry. This is a CID pointing to the identity used to sign the entry.

- 1. **sig** (bytes)
  - The signature from signing the data field or its hash with the identity referenced by the **from** field.

- 2. **data** (bytes)
  - The encapsulated entry data for easy verification of the signature.

  ###### see cbor [major types](https://www.rfc-editor.org/rfc/rfc8949.html#section-3.1) and [tag 42](https://github.com/ipld/cid-cbor/)

```
  {
    from: tag 42
    sig: type 2
    data: type 2
  }
```

#### De-encapsulate Entry Data

- 0. **v** (uint64)
  - Denotes the version of the entry. In this case equal to `3` to denote version 3.

- 1. **tag** (bytes)
  - The tag field is used to associate the entry with a specific log/database.

- 2. **clock** (uint64)
  - The clock field is a logical timestamp for entries used as a part of ordering.

- 3. **payload** (any)
  - The payload field contains the data being appended to the log. When used by OrbitDB it is the database operation/s.

- 4. **next** (CID[])
  - The next field is an array of entry CID not yet referenced by other known entries in the log, also know as log heads. Used for ordering traversal.

- 5. **refs** (CID[])
  - The refs field is an array of entry CID, like the next field, but they reference entries farther back in the log. Used for replication, not used for ordering.

```
  {
    v: type 0
    tag: type 2
    clock: type 0
    payload: type 0-5 | tag 42
    next: type 4<tag 42>
    refs: type 4<tag 42>
  }
```

#### Example Entry:

- ###### signed entry
```
  {
    from: CID(bafyreicl6ujc6ncfktctxxroxognfn7d2fqavvrryoc2lv6m4i6hpbkfti),
    sig: Uint8Array(70) [
       48,  68,   2,  32,  98, 244, 207, 200, 184, 243, 204,
        1,  40,  59,  37, 234, 179, 238, 178, 149,  97,  75,
      176, 250, 168, 189,  32, 240,  38, 193,  72, 122, 230,
       99,  18,  17,   2,  32, 124, 228,  21, 189, 116,  35,
      182, 109, 105,  83,  56, 193, 113,  34, 233,  55,  37,
      159, 119, 209, 232, 100, 148, 211,  20, 100,  54, 240,
      149, 159, 204, 198
    ],
    data: Uint8Array(77) [
      166,  97, 118,   3,  99, 116,  97, 103, 216,  42,  88,  37,
        0,   1, 113,  18,  32,  75, 245,  18,  47,  52,  69,  84,
      197,  59, 222,  46, 187, 140, 210, 183, 227, 209,  96,  10,
      214,  49, 195, 133, 165, 215, 204, 226,  60, 119, 133,  69,
      154, 100, 110, 101, 120, 116, 128, 100, 114, 101, 102, 115,
      128, 101,  99, 108, 111,  99, 107,   0, 103, 112,  97, 121,
      108, 111,  97, 100, 246
    ]
  }
```
- ###### entry data
```
  {
    v: 3,
    tag: Uint8Array(36) [
        1, 113,  18,  32,  75, 245,  18,  47,
       52,  69,  84, 197,  59, 222,  46, 187,
      140, 210, 183, 227, 209,  96,  10, 214,
       49, 195, 133, 165, 215, 204, 226,  60,
      119, 133,  69, 154
    ],
    clock: 0,
    payload: null,
    next: [],
    refs: []
  }
```

The length of an encoded entry with empty payload and reference fields is ~200 bytes.

Fields that can change in length include the clock, payload, and reference fields. Each CID added to a reference field would increase the length of the entry by ~40 bytes.

## Signature Creation and Verification

The **from** field must reference data that can be used to verify the validity of the entry. In most cases this is an [identity] with a public key.

The **sig** field must be the byte output of the DSA used to sign the **data** field with the identity referenced by the **from** field. The identity specifies the DSA and keys being used to create and verify the **sig** field.

The **data** field must be the [dag-cbor](https://github.com/ipld/ipld/blob/master/specs/codecs/dag-cbor/spec.md) encoded entry data in its bytes format.

## Logical Clock

The **clock** field is used to add more richness to ordering the log. When creating new entries:

 - If no other entries are known to exist as part of the log, the entry.clock is set to 0.
 - If entries are known to exist as part of this log, the entry.clock is set to the highest known clock value in the log + 1.

## Reference Fields

The **next** and **refs** fields reference previous entries in the log. This allows for partial order and traversal via the causal links. They have two distinct reasons for existing.

The **next** field must reference all entries that aren't referenced by other known entries. These are seen as log heads and should be the latest entries appended. New entries added to the log must reference these heads, and then become the new log heads. Entries added without knowledge of previous entries will not reference any entries and the field will be an empty array.

The **refs** field references entries further back in the log and exists to speed up fetching large parts of the log; they must not contain any entry CID already in the **next** field. Currently in the official implementation, **refs** references entries that are a power of 2 distance away when ordered as part of the log. Unlike the **next** field these references do not affect causal ordering.

Both fields arrays must be sorted greatest to least to reduce non-determinism.
