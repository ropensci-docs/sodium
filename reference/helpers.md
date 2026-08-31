# Sodium Utilities

The functions `bin2hex` and `hex2bin` convert between binary (raw)
vectors and corresponding string in hexadecimal notation. The `random`
function generates `n` crypto secure random bytes.

## Usage

``` r
bin2hex(bin)

hex2bin(hex, ignore = ":")

random(n = 1)
```

## Arguments

- bin:

  raw vector with binary data to convert to hex string

- hex:

  a string with hexadecimal characters to parse into a binary (raw)
  vector.

- ignore:

  a string with characters to ignore from `hex`. See example.

- n:

  number of random bytes or numbers to generate

## Examples

``` r
# Convert raw to hex string and back
test <- charToRaw("test 123")
x <- bin2hex(test)
y <- hex2bin(x)
stopifnot(identical(test, y))
stopifnot(identical(x, paste(test, collapse = "")))

# Parse text with characters
x2 <- paste(test, collapse = ":")
y2 <- hex2bin(x2, ignore = ":")
stopifnot(identical(test, y2))
```
