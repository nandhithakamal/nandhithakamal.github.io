---
layout: post
title: "How to TPM - Part 2 : TPM Software Stack"
author: Nandhitha Kamal
categories: blog tech
tags: tpm linux security
date: 2020-07-11
---

*This post is part of the How to TPM series. In case you haven't read the previous post(s) in this series, you might want to start from there.*

***
![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/pny68p7sdp6ws5ocafbd.jpg)

The last post covered the basics of TPM vocabulary. This post will delve into the TPM ecosystem. The Trusted Computing Group(TCG) has put together specifications for the various components that come together to create the TPM 2 Software Stack. There are multiple libraries that implement these specs to suit specific operating systems and architectures.

#### SAPI - System API
The System API  is a layer of the overall TSS architecture that provides access to all the functionality of a TPM 2.0 implementation.  It is designed to be used wherever low level calls to the TPM functions are made: firmware, BIOS, applications, OS,etc.

#### ESAPI - Extended System API
ESAPI is the layer on top of SAPI. This provides for enhanced context management and Cryptographic operations.
Even though ESAPI provides an easier way to use the TPM, using the ESAPI layer of APIs still requires an in-depth understanding of the TPM's internal workings.

#### FAPI - Feature API
The FAPI layer is on top of the ESAPI layer. FAPI provides a high-level interface including a policy definition language and key store. A user would want to create keys without any knowledge of the underlying system. This is possible by using the FAPI layer of APIs.

#### TCTI - TPM Command Transmission Interface
The TCTI is an IPC abstraction layer used to send commands to and receive responses from the TPM or the TAB/RM. This provides multiple interfaces between SAPI and the lower hardware layers depending on the type of TPM (physical TPM, tpm simulator, etc.) being used

**tpm2-tss** is a system utility that allows access to the TPM  from the OS and other programs. This library consists of implementations for all the layers from FAPI to the TCTI. [Install tpm2-tss.](https://github.com/tpm2-software/tpm2-tss/blob/master/INSTALL.md)


#### TAB/RM: This is the TPM's Access Broker and Resource Manager.
The access broker manages synchronization between the processes that use TPM simultaneously. The TAB is responsible for guaranteeing that a process using the TPM is able to proceed with the desired operation without any interference from any of the other simultanously TPM-accessinfg processes.

The resource manager manages the TPM context in a fashion similar to the virtual memory manager. The TPM has a small memory capacity and is easily constrained by how many resources can be loaded simulatenously. The resource manager is responsible for swapping in and out of memory the various resources required by a process.

In Unix, hardware devices are treated as files and can be accessed using file paths. Likewise, the tpm hardware can be accessed either using `/dev/tpm0` or using `/dev/tpmrm0`.

`/dev/tpm0` is the direct path to the tpm hardware and at any given time, only a single process can access the tpm using this path. It is very common for multiple processes to require access to tpm. For this purpose, the `/dev/tpmrm0` is used. This is the tpm's resource manager and using this multiple processes can use the tpm simultaneously.

The **tpm2-abrmd** is a system daemon that implements the TAB (TPM2 Access Broker) and resource manager specifications. The recent versions of kernel (starting from 4.12) have an in-kernal resource manager. If you are on a later version of the kernel, you would not need the tpm2-abrmd and can proceed using the in-kernal rm.
[Install tpm2-abrmd.](https://github.com/tpm2-software/tpm2-abrmd/blob/master/INSTALL.md)

#### TPM Device Driver
 The TPM device driver is an OS specific driver. It is responsible for establishing connection to the TPM and for reading from and writing to the TPM.

#### tpm2-tools
This is not part of the TPM2 Software Stack per se, but is an end-user command line tool that can be used to generate keys, view capabilities, sign, hash, unseal, etc.
[Install tpm2-tools.] (https://github.com/tpm2-software/tpm2-tools/blob/master/doc/INSTALL.md)

![Alt Text](https://dev-to-uploads.s3.amazonaws.com/i/47ftwdo73rmoce5y4zkv.jpg)
