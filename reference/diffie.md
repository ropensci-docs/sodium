# Diffie-Hellman

The Diffie-Hellman key exchange method allows two parties that have no
prior knowledge of each other to jointly establish a shared secret key
over an insecure channel. This key can then be used to encrypt
subsequent communications using a symmetric key cipher.

## Usage

``` r
diffie_hellman(key, pubkey)
```

## Arguments

- key:

  your private key

- pubkey:

  other person's public key

## Value

Returns a shared secret key which can be used in e.g.
[data_encrypt](https://docs.ropensci.org/sodium/reference/symmetric.md).

## Details

Encryption methods as implemented in
[data_encrypt](https://docs.ropensci.org/sodium/reference/symmetric.md)
require that parties have a shared secret key. But often we wish to
establish a secure channel with a party we have no prior relationship
with. Diffie-hellman is a method for jointly agreeing on a shared secret
without ever exchanging the secret itself. Sodium implements
[Curve25519](https://en.wikipedia.org/wiki/Curve25519), a
state-of-the-art Diffie-Hellman function suitable for a wide variety of
applications.

The method consists of two steps (see examples). First, both parties
generate a random private key and derive the corresponding public key
using [pubkey](https://docs.ropensci.org/sodium/reference/keygen.md).
These public keys are not confidential and can be exchanged over an
insecure channel. After the public keys are exchanged, both parties will
be able to calculate the (same) shared secret by combining his/her own
private key with the other person's public key using diffie_hellman.

After the shared secret has been established, the private and public
keys are disposed, and parties can start encrypting communications based
on the shared secret using e.g.
[data_encrypt](https://docs.ropensci.org/sodium/reference/symmetric.md).
Because the shared secret cannot be calculated using only the public
keys, the process is safe from eavesdroppers.

## References

<https://doc.libsodium.org/advanced/scalar_multiplication.html>

## Examples

``` r
# Bob generates keypair
bob_key <- keygen()
bob_pubkey <- pubkey(bob_key)

# Alice generates keypair
alice_key <- keygen()
alice_pubkey <- pubkey(alice_key)

# After Bob and Alice exchange pubkey they can both derive the secret
alice_secret <- diffie_hellman(alice_key, bob_pubkey)
bob_secret <- diffie_hellman(bob_key, alice_pubkey)
stopifnot(identical(alice_secret, bob_secret))
```
