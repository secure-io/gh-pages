---
title: 'Introduction'
date: 2019-02-11T19:27:37+10:00
weight: 2
---

Secure I/O is an open-source project that provides provable secure
authenticated encryption for I/O streams.

It is not a comprehensive cryptographic library. Instead, it is focused on one
specific problem, i.e. encrypting data streams, and solves that in a provable
secure way. Therefore, Secure I/O provides packages for various programming
languages with an easy-to-use and idiomatic API for encrypting and decrypting
I/O operations.

#### Supported Languages

At the moment there are a packages for the following languages:

 - [Go](https://github.com/secure-io/sio-go) (no stable API, yet)
 - [Rust](https://github.com/secure-io/sio-rs) (work in progress)

Secure I/O uses semantic versioning for all its packages. In particular, all
versions < 1.0.0 must be considered as not stable and may break dependent code
without any warnings. However, we try to avoid major breaking changes without 
very good reasons.

If you are missing a package for your favourite programming language consider
opening an [issue](https://github.com/secure-io/sio/issues). However, due
to limitation of resources and since we try to achieve high code quality, we
may not be able to provide and maintain such a package - in particular, if
there is no maintainer who is familiar with the language. **Contributions are
always welcome!** 

