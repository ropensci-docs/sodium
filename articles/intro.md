# Sodium: A Modern and Easy-to-Use Crypto Library

The sodium R package provides bindings to
[libsodium](https://libsodium.gitbook.io/): a modern, easy-to-use
software library for encryption, decryption, signatures, password
hashing and more.

The goal of Sodium is to provide the core operations needed to build
higher-level cryptographic tools. It is not intended for implementing
standardized protocols such as TLS, SSH or GPG. Sodium only supports a
limited set of state-of-the-art elliptic curve methods, resulting in a
simple but very powerful tool-kit for building secure applications.

|  | Authenticated Encryption | Encryption only | Authentication only |
|---:|:--:|:--:|:--:|
| **Symmetric** *(secret key)*: | [`data_encrypt` / `data_decrypt`](#secret-key-encryption) |  | [`data_tag`](#secret-key-authentication) |
| **Asymmetric** *(public+private key)*: | [`auth_encrypt` / `auth_decrypt`](#public-key-authenticated-encryption) | [`simple_encrypt` / `simple_decrypt`](#public-key-encryption) | [`sig_sign` / `sig_verify`](#public-key-authentication-signatures) |
|  |  |  |  |

### Using Sodium

All Sodium functions operate on binary data, called ‘raw’ vectors in R.
Use `charToRaw` and `rawToChar` to convert between strings and raw
vectors. Alternatively `hex2bin` and `bin2hex` can convert between
binary data to strings in hex notation:

``` r

test <- hash(charToRaw("test 123"))
str <- bin2hex(test)
print(str)
```

    [1] "e8b785b02e702c0b7edc9683130db36c91e0241ba0c489ff1e20cbb4fa3920f9"

``` r

hex2bin(str)
```

     [1] e8 b7 85 b0 2e 70 2c 0b 7e dc 96 83 13 0d b3 6c 91 e0 24 1b a0 c4 89 ff 1e
    [26] 20 cb b4 fa 39 20 f9

### Random data generator

The [`random()`](https://docs.ropensci.org/sodium/reference/helpers.md)
function generates n bytes of unpredictable data, suitable for creating
secret keys.

``` r

secret <- random(8)
print(secret)
```

    [1] 4d c6 fc 6b 9a 47 d3 1e

Implementation is platform specific, see the
[docs](https://libsodium.gitbook.io/doc/generating_random_data) for
details.

### Hash functions

Sodium has several hash functions including
[`hash()`](https://docs.ropensci.org/sodium/reference/hash.md),
[`shorthash()`](https://docs.ropensci.org/sodium/reference/hash.md),
[`sha256()`](https://docs.ropensci.org/sodium/reference/hash.md),
`sha512` and
[`scrypt()`](https://docs.ropensci.org/sodium/reference/hash.md). The
generic [`hash()`](https://docs.ropensci.org/sodium/reference/hash.md)
is usually recommended. It uses
[blake2b](https://doc.libsodium.org/hashing/generic_hashing.html) with a
configurable size between 16 bytes (128bit) and 64 bytes (512bit).

``` r

# Generate keys from passphrase
passphrase <- charToRaw("This is super secret")
hash(passphrase)
```

     [1] 98 5c 9b b6 f6 92 d5 26 10 80 99 25 3e a5 a6 66 67 13 fd 88 10 b6 12 74 86
    [26] c8 e9 5c 44 07 45 f5

``` r

hash(passphrase, size = 16)
```

     [1] eb 6c df 04 18 40 16 28 c1 b0 2e 76 f3 e6 bd 89

``` r

hash(passphrase, size = 64)
```

     [1] d0 89 68 30 26 1d 1b 85 76 dc ad 20 c9 58 0a fb b1 d0 62 ba 10 d6 80 f6 cb
    [26] c6 ae 2d 42 57 ee a0 65 fd b0 e8 90 02 ae b3 e0 4f 88 df ba ea 26 bb 47 3f
    [51] 29 5a a4 06 cd b8 05 78 83 31 66 dc 7b 24

The [`shorthash()`](https://docs.ropensci.org/sodium/reference/hash.md)
function is a special 8 byte (64 bit) hash based on
[SipHash-2-4](https://doc.libsodium.org/hashing/short-input_hashing.html).
The output of this function is only 64 bits (8 bytes). It is useful for
in e.g. Hash tables, but it should not be considered
collision-resistant.

### Secret key encryption

Symmetric encryption uses the same secret key for both encryption and
decryption. It is mainly useful for encrypting local data, or as a
building block for more complex methods.

Most encryption methods require a `nonce`: a piece of non-secret unique
data that is used to randomize the cipher. This allows for safely using
the same `key` for encrypting multiple messages. The nonce should be
stored or shared along with the ciphertext.

``` r

key <- hash(charToRaw("This is a secret passphrase"))
msg <- serialize(iris, NULL)

# Encrypt with a random nonce
nonce <- random(24)
cipher <- data_encrypt(msg, key, nonce)

# Decrypt with same key and nonce
orig <- data_decrypt(cipher, key, nonce)
identical(iris, unserialize(orig))
```

    [1] TRUE

Because the secret has to be known by all parties, symmetric encryption
by itself is often impractical for communication with third parties. For
this we need asymmetric (public key) methods.

### Secret key authentication

Secret key authentication is called tagging in Sodium. A tag is
basically a hash of the data together with a secret key.

``` r

key <- hash(charToRaw("This is a secret passphrase"))
msg <- serialize(iris, NULL)
mytag <- data_tag(msg, key)
```

To verify the integrity of the data at a later point in time, simply
re-calculate the tag with the same key:

``` r

stopifnot(identical(mytag, data_tag(msg, key)))
```

The secret key protects against forgery of the data+tag by an
intermediate party, as would be possible with a regular checksum.

### Public key encryption

Where symmetric methods use the same secret key for encryption and
decryption, asymmetric methods use a key-pair consisting of a public key
and private key. The private key is secret and only known by its owner.
The public key on the other hand can be shared with anyone. Public keys
are often published on the user’s website or posted in public
directories or keyservers.

``` r

key <- keygen()
pub <- pubkey(key)
```

In public key encryption, data encrypted with a public key can only be
decrypted using the corresponding private key. This allows anyone to
send somebody a secure message by encrypting it with the receivers
public key. The encrypted message will only be readable by the owner of
the corresponding private key.

``` r

# Encrypt message with pubkey
msg <- serialize(iris, NULL)
ciphertext <- simple_encrypt(msg, pub)

# Decrypt message with private key
out <- simple_decrypt(ciphertext, key)
stopifnot(identical(out, msg))
```

### Public key authentication (signatures)

Public key authentication works the other way around. First, the owner
of the private key creates a ‘signature’ (an authenticated checksum) for
a message in a way that allows anyone who knows his/her public key to
verify the integrity of the message and identity of the sender.

Currently sodium requires a different type of key-pair for signatures
(ed25519) than for encryption (curve25519).

``` r

# Generate signature keypair
key <- sig_keygen()
pubkey <- sig_pubkey(key)

# Create signature with private key
msg <- serialize(iris, NULL)
sig <- sig_sign(msg, key)
print(sig)
```

     [1] 78 92 60 9b 6d 72 6b 5d b2 c6 a5 a7 fa 69 f0 86 0f 3f 10 90 11 c1 0b da 1d
    [26] af 17 13 cc 01 84 40 b1 4a a6 e0 da 11 03 e6 90 e6 0d 47 03 d2 1f b6 1b 4c
    [51] e2 b3 a0 1c f1 d9 de ca fa fd 53 d1 a6 03

``` r

# Verify a signature from public key
sig_verify(msg, sig, pubkey)
```

    [1] TRUE

Signatures are useful when the message itself is not confidential but
integrity is important. A common use is for software repositories where
to include an index file with checksums for all packages, signed by the
repository maintainer. This allows client package managers to verify
that the binaries were not manipulated by intermediate parties during
the distribution process.

### Public key authenticated encryption

Authenticated encryption implements best practices for secure messaging.
It requires that both sender and receiver have a keypair and know each
other’s public key. Each message gets authenticated with the key of the
sender and encrypted with the key of the receiver.

``` r

# Bob's keypair:
bob_key <- keygen()
bob_pubkey <- pubkey(bob_key)

# Alice's keypair:
alice_key <- keygen()
alice_pubkey <- pubkey(alice_key)

# Bob sends encrypted message for Alice:
msg <- charToRaw("TTIP is evil")
ciphertext <- auth_encrypt(msg, bob_key, alice_pubkey)

# Alice verifies and decrypts with her key
out <- auth_decrypt(ciphertext, alice_key, bob_pubkey)
stopifnot(identical(out, msg))

# Alice sends encrypted message for Bob
msg <- charToRaw("Let's protest")
ciphertext <- auth_encrypt(msg, alice_key, bob_pubkey)

# Bob verifies and decrypts with his key
out <- auth_decrypt(ciphertext, bob_key, alice_pubkey)
stopifnot(identical(out, msg))
```

Note that even though public keys are not confidential, you should not
exchange them over the same insecure channel you are trying to protect.
If the connection is being tampered with, the attacker could simply
replace the key with another one to hijack the interaction.
