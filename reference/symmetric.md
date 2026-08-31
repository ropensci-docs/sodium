# Symmetric Encryption and Tagging

Encryption with authentication using a 256 bit shared secret. Mainly
useful for encrypting local data. For secure communication use
public-key encryption
([simple_encrypt](https://docs.ropensci.org/sodium/reference/simple.md)
and
[auth_encrypt](https://docs.ropensci.org/sodium/reference/messaging.md)).

## Usage

``` r
data_encrypt(msg, key, nonce = random(24))

data_decrypt(bin, key, nonce = attr(bin, "nonce"))

data_tag(msg, key)
```

## Arguments

- msg:

  message to be encrypted

- key:

  shared secret key used for both encryption and decryption

- nonce:

  non-secret unique data to randomize the cipher

- bin:

  encrypted ciphertext

## Details

Symmetric encryption uses a secret key to encode and decode a message.
This can be used to encrypt local data on disk, or as a building block
for more complex methods.

Because the same `secret` is used for both encryption and decryption,
symmetric encryption by itself is impractical for communication. For
exchanging secure messages with other parties, use asymmetric
(public-key) methods (see
[simple_encrypt](https://docs.ropensci.org/sodium/reference/simple.md)
or
[auth_encrypt](https://docs.ropensci.org/sodium/reference/messaging.md)).

The `nonce` is not confidential but required for decryption, and should
be stored or sent along with the ciphertext. The purpose of the `nonce`
is to randomize the cipher to protect against re-use attacks. This way
you can use one and the same secret for encrypting multiple messages.

The data_tag function generates an authenticated hash that can be stored
alongside the data to be able to verify the integrity of the data later
on. For public key signatures see `sig_sign` instead.

## References

<https://libsodium.gitbook.io/doc/public-key_cryptography/authenticated_encryption>

## Examples

``` r
# 256-bit key
key <- sha256(charToRaw("This is a secret passphrase"))
msg <- serialize(iris, NULL)

# Encrypts with random nonce
cipher <- data_encrypt(msg, key)
orig <- data_decrypt(cipher, key)
stopifnot(identical(msg, orig))

# Tag the message with your key (HMAC)
tag <- data_tag(msg, key)
```
