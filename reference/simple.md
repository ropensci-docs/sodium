# Anonymous Public-key Encryption (Sealed Box)

Create an encrypted message (sealed box) from a curve25519 public key.

## Usage

``` r
simple_encrypt(msg, pubkey)

simple_decrypt(bin, key)
```

## Arguments

- msg:

  message to be encrypted

- pubkey:

  public key of the receiver

- bin:

  encrypted ciphertext

- key:

  private key of the receiver

## Details

Simple public key encryption allows for sending anonymous encrypted
messages to a recipient given its public key. Only the recipient can
decrypt these messages, using its private key.

While the recipient can verify the integrity of the message, it cannot
verify the identity of the sender. For sending authenticated encrypted
messages, use
[auth_encrypt](https://docs.ropensci.org/sodium/reference/messaging.md)
and
[auth_decrypt](https://docs.ropensci.org/sodium/reference/messaging.md).

## References

<https://doc.libsodium.org/public-key_cryptography/sealed_boxes.html>

## Examples

``` r
# Generate keypair
key <- keygen()
pub <- pubkey(key)

# Encrypt message with pubkey
msg <- serialize(iris, NULL)
ciphertext <- simple_encrypt(msg, pub)

# Decrypt message with private key
out <- simple_decrypt(ciphertext, key)
stopifnot(identical(out, msg))
```
