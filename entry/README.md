![](https://img.shields.io/badge/status-wip-orange.svg?style=flat-square)

# Entry

-----

**Abstract**

Entries are immutable and reference previous entries by [content-address](https://docs.ipfs.io/concepts/content-addressing) also called CID; creating causal and traversable links.

Beside references to previous entries, entries include other information:

 - version
 - database tag
 - logical clock
 - payload
 - identity

# Table of Contents

- [Introduction](#introduction)
- [Interface](#interface)
- [Validity](#validity)
- [Reference Fields](#reference-fields)
- [Ordering](#ordering)

## Introduction

Entries are used to append updates in an immutable and conflict free way. Peers share them with each other and add them to their local replica.
Before new entries can be checked for access and added to the replica; their tag, digital signature, and writer identity need to be verified.

## Interface

The entry interface is defined:

###### see cbor [encoding specification](https://www.rfc-editor.org/rfc/rfc8949.html#name-specification-of-the-cbor-e), [tag 42](https://github.com/ipld/cid-cbor/), and [identity interface](../identity#Interface)

```
{
  cid: tag 42
  tag: type 2
  clock: type 0
  identity: identity
  sig: type 2
  payload: any
  next: type 4<tag 42>
  refs: type 4<tag 42>
}
```

Entry formats do not need to match the interface at the encoding level. When decoded and verified by the entry format, this interface must be exposed.

## Validity

For an entry to be a valid the interface must be matched, the tag must belong to the log, and signature must be verified. A verified signature has a signature that was created by the identity signing the entry data.

All fields, excluding `identity` and `sig`, must be digitally signed. The signature must be created using the private key that is the pair of the identity's public key.

If an entry is not valid it must be expelled and must not be traversed for references.

Entry validity does not have to do with, and is prior to access control.

## Logical Clock

The **clock** field is used to add more richness to ordering the log. When creating new entries:

 - If no other entries are known to exist as part of the log, the entry.clock is set to 0.
 - If entries are known to exist as part of this log, the entry.clock is set to the highest known clock value + 1.

## Reference Fields

The **next** and **refs** fields reference previous entries in the log. This allows for partial order and traversal via the causal links. They have two distinct reasons for existing.

The **next** field must reference all entries that aren't referenced by other known entries. These are seen as log heads and should be the latest entries appended. New entries added to the log must reference these heads, and then become the new log heads. Entries added without knowledge of previous entries will not reference any entries and the field will be an empty array.

The **refs** field references entries further back in the log and exists to speed up fetching large parts of the log; they must not contain any entry CID already in the **next** field. Currently in the official implementation, **refs** references entries that are a power of 2 distance away when ordered as part of the log. Unlike the **next** field these references do not affect causal ordering.

Both fields arrays must be sorted greatest to least to reduce non-determinism.

## Ordering

Entries ensure total order without conflict. There are no duplicate entries, they are deduplicated by their hash, and every builds on previous ones.

There are a few ordering metrics that can be used. The heaviest metric is the causal relationship between entries. The subsequest weights are the `entry.identity.id` and the `entry.hash`.

`causality > id > hash`

TODO: define ording to spec, keep causal order always, other two could be messed with by the peer
