---
layout: post
title: "How to TPM - Part 1: TPM Vocabulary"
author: Nandhitha Kamal
categories: blog tech
tags: tpm linux security
date: 2020-05-27
---

![How to TPM](https://dev-to-uploads.s3.amazonaws.com/i/sdrnihmljg37ycb8v5qz.jpg)
As part of a recent piece of work, we did a spike on TPM. During the process, I realised that there wasn't material available for beginners out there, at least not in a single place. A lot of the documentation around it is cryptic (pun unintentional!). The idea is to make it easier to understand for TPM novices like me. I plan on documenting this in parts, so that both the reading and writing are not overwhelming. Also, this series will not delve into the technical implementation of the specification and is my basic understanding of TPM and how one can use the TPM.

### TPM - A Brief Introduction

#### What is a TPM?
TPM stands for Trusted Platform Module. TPM is a cryptographic co-processor. What this means is that it the TPM in addition to other things, contains a processor that is specialised for cryptographic use-cases. It is a hardware chip that is attached to the motherboard of a computer. The TPM is an inbuilt security module.
Over time, there have been multiple versions of TPM 
- TPM 1.1b 
- TPM 1.2
- TPM 2

In this post I shall be talking only about TPM 2, unless I want to delve into some historical context or differences between the multiple versions.

#### What can a TPM do?
 The use-cases I have explored so far, TPM can be used to 
- Generate key pairs
- Store key pairs
- Hash
- Sign
- Encrypt/Decrypt
- Verify Signature
- Generate Random Numbers

It can likely be used for a whole bunch of other use-cases as well. I haven't explored other things at this point.
Now, moving on to TPM specific vocabulary.

##### Hierarchies
You can think of a hierarchy as a tree structure with the parent key at the root. This wraps around and encrypts all of its children keys. Keys can be created under any of the following 4 hierarchies.
- **Endorsement** - The endorsement hierarchy is the privacy-sensitive tree and is the hierarchy of choice when the user has privacy concerns. TPM and platform vendors certify that primary keys in this hierarchy are constrained to an authentic TPM attached to an authentic platform.
- **Owner** - Also called storage hierarchy, this is intended to be used by the platform owner, i.e., the device owner or user
- **Platform** - Intended to be under the control of the platform manufacturer. This is used by the BIOS and System Management Mode (SMM).
- **Null**

Of the above, Endorsement, Owner and Platform hierarchies are persistent hierarchies. And the Null hierarchy is an ephemeral hierarchy. What this means is that, keys created using either of Endorsement/Owner/Platform hierarchies can survive system re-boots. Keys created using Null hierarchy on the other hand are ephemeral i.e., will not be persisted between system reboots. All TPM objects (keys/data) belong to a hierarchy.
The cryptographic root of each hierarchy is a seed. A seed is a large random number that the TPM generates. This seed is never exposed outside the TPM's secure boundary. This seed is the starting value from which keys are generated. It is also possible to further add security to the keys by adding password to the keys. Usage of the keys would now require the key password also.

Although the TPM comes with a limited amount of Non-Volatile(NV) memory, it is possible to generate more than just a few keys. This is owing to the fact that one can recreate the same keys by applying a cryptographic algorithm on a given seed. It is deterministic. Thus the keys need not be saved in the file system(TPM allows for saving the keys in file system); it is always possible to regenerate them. 

Null hierarchy is an exception to this. For the ephemeral Null hierarchy, this seed value changes between power cycles. Once the seed value changes, it would not be possible to regenerate the previous keys. For this reason, the null hierarchy can be used to create password-less, transient session keys.

##### Handles
A handle is a 32-bit reference to an entity in the TPM. An identifier that uniquely identifies a TPM resource that occupies TPM memory.
There are 3 types of key handles. 
- Transient
- Persistent
- Permanent

Transient handles are addresses of keys in the NVRAM. These point to keys that have either been loaded using tpm2_load or keys that have been (re)created but not persisted.

Persistent handles reference keys in the NV memory of the TPM. These keys are persisted through power cycles and hence need not be recreated. To use the keys, they first need to loaded into the TPM. However, with the persistent keys, one need not load as these are resident in the TPM itself. If the key (re)creation time is longer, they are persisted using tpm2_evictcontrol. Otherwise, they can be recreated upon requirement.
Permanent handles reference entities that can't be deleted. Among other things, this also includes handles to the 4 hierarchies.
In addition there are other types of handles for NV indices, saved session, PCR, etc.

#### Creating Keys
To create keys for signing, one needs to first create a primary key under any of the 4 hierarchies. This primary key can now be used to create children keys. The children keys can now be used for signing. It is also possible for each child key of the primary key to become parent keys if it meets certain conditions. Thus one can have a deep hierarchy of keys.
The primary key cannot sign. It can only decrypt the child key for signing. This is specified by what is called key attributes. The key's attributes are set at key creation. These attributes are used to control the behaviour of the keys.

Some of the commonly used attributes are:
- **fixedTPM:** If this attribute is set, the key cannot be duplicated in any way
- **fixedParent:** If this attribute is set, the parent of this key cannot be changed (more info on this in the future posts).
- **sign:** If a key is to be used for signing, this attribute needs to be set.
- **restricted:** A key with this attribute set can sign only hashes generated within the TPM
- **decrypt:** Only keys with this attribute are allowed to create or load children key. If this is attribute is set, the key can be a parent key.
- **sensitiveDataOrigin:** If this attribute is set, it indicates that the TPM generated all the sensitive data other than auth value for this key
- **noDA:** The object is subject to dictionary attack protections

By using a combination of these attributes, one can create primary keys, keys for signing, parent keys, etc.

In the next parts, I shall cover the various TPM components, accessing the TPM, getting hands-on with creating keys and signing.
