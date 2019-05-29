---
title: 'FAQ'
date: 2019-02-11T19:27:37+10:00
weight: 3
---

<h4 id="why-should-i-use-secure-io">
    <a href="/doc/faq#why-should-i-use-secure-io" style="text-decoration:none">
        Why should I use Secure I/O?
    </a>
</h4>

Actually, there are many different ways to encrypt data. For example, a block
cipher, e.g.
[AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard), in counter
mode ([CTR](https://en.wikipedia.org/wiki/CTR_mode#Counter_(CTR))) has been and
still is a quite popular encryption scheme. Even though, AES-CTR preserves the
confidentiality of the encrypted data, it provides no integrity protection
whatsoever. In particular, an attacker can flip a bit of the ciphertext and
therefore, can cause a flip of the corresponding plaintext bit during
decryption. AES-CTR cannot detect this and won't return an error. This is
indeed a real problem because an attacker often knows or can guess the
structure and/or parts of the encrypted plaintext.

Therefore, some applications have switched to authenticated encryption
([AEAD](https://en.wikipedia.org/wiki/Authenticated_encryption)) schemes, like
[AES-GCM](https://en.wikipedia.org/wiki/Galois/Counter_Mode). However, AEAD
(on its own) does not solve all security issues w.r.t. data integrity. In
particular, an AEAD only preserves confidentiality and integrity in the
*atomic message setting* but not when data is processed as stream. For more
details about using AEAD schemes for streaming take a look at this well-written
[blog post](https://www.imperialviolet.org/2014/06/27/streamingencryption.html) by
[Adam Langley](https://twitter.com/agl__). 

Now, why should you use the scheme implemented by Secure I/O instead of e.g.
AES-GCM?

Well, actually you can use Secure I/O with AES-GCM. In fact, Secure I/O
implements an encryption scheme (*secure channel construction*) on top of AEAD.
The channel construction splits the plaintext into fixed-size chunks
(*fragments*) and encrypts and authenticates resp. decrypts and verifies each
fragment separately using the AEAD scheme.

Now, assuming the AEAD scheme is "secure", it can be proven that the channel
construction implemented by Secure I/O is "secure" as well. Take a look at
[What does *provable secure* mean?](/doc/faq#what-does-provable-secure-mean) if
you want to learn more about this.

Also, invoking the AEAD scheme once per fragment instead of only once for the
entire plaintext may sound like a lot of computational overhead. However, this
is not the case. The channel construction is almost as performant as the AEAD
scheme itself. In particular, encrypting plaintexts > 1 MB with the channel
construction is just 1-3% slower than using AES-GCM directly.  

<h4 id="what-does-provable-secure-mean">
    <a href="/doc/faq#what-does-provable-secure-mean" style="text-decoration:none">
        What does <i>provable secure</i> mean?
    </a>
</h4>

<h4 id="what-is-a-nonce">
    <a href="/doc/faq#what-is-a-nonce" style="text-decoration:none">
        What's a <i>nonce</i>?
    </a>
</h4>

Nonce stands for: <i><b>n</b>umber used <b>once</b></i> - and that's what it
is. It's a value that must only be used once. Usually, in the context of
encryption, a nonce is associated with a secret key.  So, to be precise, not
the nonce must be unique but the combination of a secret key and a nonce.
In contrast to the secret key, it is not necessary to keep the nonce private.

Why is this important? Because using the same key-nonce combination twice,
usually breaks the security of the encryption scheme such that an attacker
can trivially decrypt ciphertexts or create forgeries.  

Therefore, I repeat: **Never use the same key-nonce combination twice!**

Easier said than done. Therefore, some recommendations for dealing with nonce
values:
 
 - Ideally, if you can, use APIs that handle nonce values internally such that
   you don't have to care.
 - As an alternative, you can use an unique secret key. Either generate it
   randomly or derive it from a master key or password using a appropriate
   function (e.g. [Argon2id](https://en.wikipedia.org/wiki/Argon2)) - depending on
   you application.  If you use unique 256 bit keys, it does not matter how you
   choose the nonce.  You can even set it to all zeros in this case. If you use
   128 bit keys you should still choose a random nonce since the probability of
   choosing a 128 key twice (at random) is not negligible.

 - If you have to use the same secret key multiple times then you have to make
   sure that you don't use the same nonce value twice. You can apply different
   strategies depending on the size of the nonce value:
    - If the nonce is ≥ than 160 bits (20 bytes) then you can choose the
      nonce value at random (`dev/urandom` is your friend 😉) 
      as long as you don't use the same secret key across millions of different
      users or applications - which you shouldn't do anyway.
    - If the nonce is ≥ 96 bits (12 bytes) then you can still choose the
      nonce at random but not too many times - not more ~4 billion times.
      That may sound a lot, but in times of cloud computing this limit can
      be reached quickly.
    - If the nonce is < 96 bits you should not choose the nonce at random.
      The best you can do is trying to change the setup such that you can 
      use unique keys. 

<h4 id="what-is-associated-data">
    <a href="/doc/faq#what-is-associated-data" style="text-decoration:none">
            What's <i>associated data</i>?
    </a>
</h4>

Modern encryption schemes encrypt and authenticate data. Usually, this happens
in two steps. First, the encryption scheme produces some ciphertext by
encrypting the plaintext. Then the scheme authenticates the ciphertext by
computing a checksum
([MAC](https://en.wikipedia.org/wiki/Message_authentication_code)) over the
ciphertext and appends this checksum to the ciphertext.

Now, associated data is data that is passed to the encryption scheme to be
authenticated but not encrypted. It is data that is associated to the encrypted
data. Whenever you want to decrypt the ciphertext you also have to provide
exactly the same associated data. Otherwise, the integrity verification of the
encryption scheme will fail and report an error.

This is useful if you want to ensure that some public data, that belongs to the
encrypted data, cannot be modified. For example, consider file encryption. Of 
course, nobody (without knowing the secret key) should be able to decrypt a file.
However, an attacker can still e.g. rename or move files to different locations.
Such an attacker can cause quite some confusion. 

Therefore, you could use either the file name or the entire (absolute) file path
as associated data. Using the file name prevents renaming the file but allows
moving the file to a different location - e.g. to another directory. In contrast, 
using the entire file path also prevents moving the file.

<h4 id="which-algorithm-should-i-use">
    <a href="/doc/faq#which-algorithm-should-i-use" style="text-decoration:none">
        Which algorithm should I use?
    </a>
</h4>

The channel construction implemented by Secure I/O demands an
[AEAD](https://en.wikipedia.org/wiki/Authenticated_encryption) scheme.
There are two well-studied and commonly used AEAD:

 - AES-GCM  
   AES-GCM supports 128 (AES-128-GCM) and 256 (AES-256-GCM) bit keys. 
   Further, it demands a 96 bit nonce. However, since Secure I/O uses
   32 bit of the nonce you can only choose 64 bit of it. As explained
   in [What's a *nonce*?](#what-is-a-nonce), you should always use an unique
   secret key if you want to use Secure I/O with AES-GCM. 
     
 - ChaCha20-Poly1305  
   ChaCha20-Poly1305 supports 256 bit keys and 96 bit nonce values. Again, since
   Secure I/O uses 32 bit of the nonce you can only choose 64 bit of it. As explained
   in [What's a *nonce*?](#what-is-a-nonce), you should always use an unique
   secret key if you want to use Secure I/O with ChaCha20-Poly1305.  
   However, there is a variant of ChaCha20-Poly1305 (XChaCha20-Poly1305) that supports
   192 bit nonce values. As mentioned above, Secure I/O reserves 32 bit such that you
   can use 160 bit of the nonce. This is sufficiently large such that you can reuse the
   secret key for randomly generated nonce values.

So, which one should I choose?

**TL;DR:** Use ChaCha20-Poly1305 - or XChaCha20-Poly1305 if you have to reuse
secret keys.  It is fast on CPUs that provide
[SIMD](https://en.wikipedia.org/wiki/SIMD) instructions and implementations are
usually side-channel-resistant.

In general, it depends on the CPU, the programming language resp. AEAD
implementation and the type of your application. It's important to know that
[AES](https://en.wikipedia.org/wiki/Advanced_Encryption_Standard) has been
standardized by the
[NIST](https://nvlpubs.nist.gov/nistpubs/FIPS/NIST.FIPS.197.pdf).  Therefore,
it is approved for financial, governmental and military use (in the U.S.).  As a
consequence, many CPU vendors have implemented (parts of) AES in hardware and
provide [special assembler
instructions](https://en.wikipedia.org/wiki/AES_instruction_set) to speed up
encryption.   
Now, AES (and AES-GCM) implementations are very fast and
side-channel-resistant on CPUs that provide special hardware support. However,
they are much slower and rarely side-channel-resistant on CPUs without such
hardware support.

In contrast, ChaCha20-Poly1305 has been developed as an alternative to AES-GCM
and has been standardized by the [IETF](https://www.ietf.org/) in [RFC
7539](https://tools.ietf.org/html/rfc7539). It is quite fast as well if the CPU
provides [SIMD](https://en.wikipedia.org/wiki/SIMD) instructions. In addition,
ChaCha20-Poly1305 is much easier to implement securely than AES-GCM.

Concluding, for certain types of applications (financial or governmental) you may
have to use AES-GCM due to legal regulations. Also, if all your CPUs provide
special hardware support for AES then AES-GCM is the straight forward solution.
Otherwise, you may want to use ChaCha20-Poly1305. Anyway, if your biggest problem
is whether you use AES-GCM or ChaCha20-Pol1305 then you don't have problems 😉.
