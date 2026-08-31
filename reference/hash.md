# Hash Functions

Functions to calculate cryptographic hash of a message, with optionally
a key for HMAC applications. For storing passwords, use
[password_store](https://docs.ropensci.org/sodium/reference/password.md)
instead.

## Usage

``` r
hash(buf, key = NULL, size = 32)

scrypt(buf, salt = raw(32), size = 32)

argon2(buf, salt = raw(16), size = 32)

shorthash(buf, key)

sha512(buf, key = NULL)

sha256(buf, key = NULL)
```

## Arguments

- buf:

  data to be hashed

- key:

  key for HMAC hashing. Optional, except for in `shorthash`.

- size:

  length of the output hash. Must be between 16 and 64 (recommended is
  32)

- salt:

  non-confidential random data to seed the algorithm

## Details

The generic `hash` function is recommended for most applications. It
uses dynamic length
[BLAKE2b](https://libsodium.gitbook.io/doc/hashing/generic_hashing)
where output size can be any value between 16 bytes (128bit) and 64
bytes (512bit).

The scrypt hash function is designed to be CPU and memory expensive to
protect against brute force attacks. This algorithm is also used by the
[password_store](https://docs.ropensci.org/sodium/reference/password.md)
function.

The argon2 hash function is also designed to be CPU and memory expensive
to protect against brute force attacks. Argon2 is a password-hashing
function that summarizes the state of the art in the design of
memory-hard functions

The `shorthash` function is a special 8 byte (64 bit) hash based on
[SipHash-2-4](https://libsodium.gitbook.io/doc/hashing/short-input_hashing).
The output of this function is only 64 bits (8 bytes). It is useful for
in e.g. Hash tables, but it should not be considered
collision-resistant.

Hash functions can be used for HMAC by specifying a secret `key`. They
key size for `shorthash` is 16 bytes, for `sha256` it is 32 bytes and
for `sha512` it is 64 bytes. For `hash` the key size can be any value
between 16 and 62, recommended is at least 32.

## References

<https://libsodium.gitbook.io/doc/hashing/generic_hashing>

## Examples

``` r
# Basic hashing
msg <- serialize(iris, NULL)
hash(msg)
#>  [1] 2f 23 f4 53 2e 17 c4 a6 dc 1c 0c 7f cd 07 58 40 1d 8b 61 8b 98 7c bb 28 9c
#> [26] 1d 33 23 d9 3e 6f 12
sha256(msg)
#>  [1] 80 c5 b2 37 01 6d 2d 03 cb 76 c9 fd e7 d7 7c be 82 3a ab 3e fd ce 7c f2 f4
#> [26] 79 03 44 19 77 3e c8
sha512(msg)
#>  [1] 3b a0 7d 32 ab 9f 6b 0d 3a 8a 03 b8 72 1e 6d 58 a2 32 7e be cb 98 2a 78 56
#> [26] 18 f5 63 62 9c 45 72 ec e6 43 f4 03 99 cc 0b ac 1b 71 dd bc e4 19 8e 0f aa
#> [51] 84 e6 91 7a e2 17 a5 96 3e d1 1f ee f4 58
scrypt(msg)
#>  [1] 04 8c 14 3e b7 9b 2a 87 cc 8b 62 4b e2 63 e1 d3 76 f7 1e 99 70 a4 72 26 e2
#> [26] 3a 6f 3e d6 62 81 16

# Generate keys from passphrase
passphrase <- charToRaw("This is super secret")
key <- hash(passphrase)
shortkey <- hash(passphrase, size = 16)
longkey <- hash(passphrase, size = 64)

# HMAC (hashing with key)
hash(msg, key = key)
#>  [1] f2 bd 0a 6a 2c fd ca 0a df c6 e8 8a cb a9 65 a8 db 0a b4 64 94 07 3c 09 16
#> [26] bc b7 78 17 8c 35 aa
shorthash(msg, shortkey)
#> [1] 97 5a a1 de ad 13 15 27
sha256(msg, key = key)
#>  [1] 21 cb 83 6b 68 6a 6d 7b 1b 48 46 d7 52 8d 39 9d 16 87 07 e4 e4 06 10 70 2d
#> [26] 48 f1 c1 d6 f6 29 89
sha512(msg, key = longkey)
#>  [1] e3 45 b9 2c 9b 45 5d c6 25 a2 ac c6 60 02 dd 7a f3 ff 5a 6b 9c c8 07 84 fe
#> [26] f1 d7 f0 14 70 cd 80 1b 55 a3 6f 4c ab f6 22 f2 33 87 18 68 52 96 ea 33 ca
#> [51] 09 fb 81 34 e5 7a 19 5a 63 31 c2 a4 bf 71
```
