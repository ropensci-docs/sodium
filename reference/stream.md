# Stream Ciphers

Generate deterministic streams of random data based off a secret key and
random nonce.

## Usage

``` r
chacha20(size, key, nonce)

xchacha20(size, key, nonce)

salsa20(size, key, nonce)

xsalsa20(size, key, nonce)
```

## Arguments

- size:

  length of cipher stream in bytes

- key:

  secret key used by the cipher

- nonce:

  non-secret unique data to randomize the cipher

## Details

You usually don't need to call these methods directly. For local
encryption use
[data_encrypt](https://docs.ropensci.org/sodium/reference/symmetric.md).
For secure communication use
[simple_encrypt](https://docs.ropensci.org/sodium/reference/simple.md)
or
[auth_encrypt](https://docs.ropensci.org/sodium/reference/messaging.md).

Random streams form the basis for most cryptographic methods. Based a
shared secret (the key) we generate a predictable random data stream of
equal length as the message we need to encrypt. Then we
[xor](https://rdrr.io/r/base/Logic.html) the message data with this
random stream, which effectively inverts each byte in the message with
probability 0.5. The message can be decrypted by re-generating exactly
the same random data stream and
[xor](https://rdrr.io/r/base/Logic.html)'ing it back. See the examples.

Each stream generator requires a `key` and a `nonce`. Both are required
to re-generate the same stream for decryption. The key forms the shared
secret and should only known to the trusted parties. The `nonce` is not
secret and should be stored or sent along with the ciphertext. The
purpose of the `nonce` is to make a random stream unique to protect
against re-use attacks. This way you can re-use a your key to encrypt
multiple messages, as long as you never re-use the same nonce.

## References

<https://libsodium.gitbook.io/doc/advanced/stream_ciphers/xsalsa20>

## Examples

``` r
# Very basic encryption
myfile <- file.path(R.home(), "COPYING")
message <- readBin(myfile, raw(), file.info(myfile)$size)
passwd <- charToRaw("My secret passphrase")

# Encrypt:
key <- hash(passwd)
nonce8 <- random(8)
stream <- chacha20(length(message), key, nonce8)
ciphertext <- base::xor(stream, message)

# Decrypt:
stream <- chacha20(length(ciphertext), key, nonce8)
out <- base::xor(ciphertext, stream)
stopifnot(identical(out, message))

# Other stream ciphers
stream <- salsa20(10000, key, nonce8)
stream <- xsalsa20(10000, key, random(24))
stream <- xchacha20(10000, key, random(24))
```
