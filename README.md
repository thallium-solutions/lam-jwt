# @lam/jwt

`@lam/jwt` provides JSON Web Token signing, verification, decoding, and JWKS key
rotation for the [Lammergeier programming language](https://github.com/thallium-solutions/lammergeier-lang).
It exposes the established `Jwt` and `JwtKeySet` APIs as a standalone package and
uses [`github.com/golang-jwt/jwt/v5`](https://github.com/golang-jwt/jwt) for the
underlying JWT and cryptographic operations.

## Features

- HS256 and HS512 signing, verification, and claim decoding.
- RS256 signing, verification, and claim decoding from PEM keys.
- Automatic `iat` and optional `exp` claims when signing.
- Configurable verification clock-skew leeway.
- `Result`-returning variants for error-aware signing and verification.
- Thread-safe `JwtKeySet` support for HMAC secrets, RSA public keys, JWKS
  documents, `kid` selection, and key rotation.
- Explicit unverified payload and `kid` helpers for diagnostics only.

## Requirements

- `lamc` 1.16.0 or newer within the compatible `1.x` series (`^1.16`).
- A Go toolchain available to `lamc` for the pinned
  `github.com/golang-jwt/jwt/v5 v5.3.1` dependency.

## Installation

Install directly from Git:

```bash
lamc install https://github.com/thallium-solutions/lam-jwt.git
```

Install from a local path while developing or using a checkout:

```bash
lamc install /path/to/lam-jwt
```

Install the published registry release:

```bash
lamc install @lam/jwt@1.0.0
```

All installation forms use the package name declared in `lamlib.toml`, so the
consumer import remains:

```lammergeier
from @lam/jwt import Jwt, JwtKeySet
```

## Basic usage

```lammergeier
from @lam/jwt import Jwt

func main() {
    claims: dict[str, any] = {}
    claims["sub"] = "alice"
    claims["role"] = "admin"

    token: str = Jwt.signHS256(claims, "replace-with-a-strong-secret", 3600)
    if not Jwt.verifyHS256(token, "replace-with-a-strong-secret") {
        panic("invalid token")
    }

    payload: dict[str, any] = Jwt.decodeHS256(token, "replace-with-a-strong-secret")
    print(payload["sub"])
}
```

Signing adds `iat` automatically. A positive `ttlSeconds` also adds `exp` at
`now + ttlSeconds`; a value of `0` or less omits expiry. Verification allows 30
seconds of clock skew by default. Change it globally with `Jwt.setLeeway(60)`
and inspect it with `Jwt.getLeeway()`.

HS512 uses `signHS512`, `verifyHS512`, and `decodeHS512`. RS256 uses
`signRS256`, `verifyRS256`, and `decodeRS256` with PEM-encoded private/public
keys.

## Result-aware verification

The `try*` methods retain error details instead of returning an empty value or
`false`. They work with Lammergeier's `?` and `do/catch` flow:

```lammergeier
from @lam/jwt import Jwt

func verifyRequest(token: str, secret: str) {
    do {
        payload: dict[str, any] = Jwt.tryDecodeHS256(token, secret)?
        print(payload["sub"])
    } catch err {
        print(f"JWT verification failed: {err}")
    }
}
```

`Jwt.trySignHS256` returns a `Result` for signing. `Jwt.tryDecodeHS256` and
`Jwt.tryDecodeRS256` return `JwtVerifyError` failures when parsing, algorithm,
signature, or claim validation fails. `JwtKeySet.tryDecodeRS256` provides the
same error-aware behavior for an RSA key set.

## JWKS and key rotation

`JwtKeySet` is a concurrent-safe verification-key collection. RSA JWKS entries
are selected by the token's `kid`; if a token has no matching `kid`, all keys
of the expected family are tried to support rotation windows.

```lammergeier
from @lam/jwt import JwtKeySet
from lamerrors import Result

func verifyOidcToken(token: str, jwksJson: str) -> bool {
    keys: JwtKeySet = JwtKeySet()
    loaded: Result = keys.loadJwks(jwksJson)
    if not loaded.ok() {
        return false
    }
    return keys.verifyRS256(token)
}
```

A JWKS document must contain a `keys` array. RSA entries need `kty = "RSA"`, a
non-empty `kid`, and base64url-encoded modulus (`n`) and exponent (`e`). Other
key types are skipped. PEM public keys can be added individually with
`addRSA(kid, publicKeyPem)`.

HMAC keys can also rotate by `kid`:

```lammergeier
from @lam/jwt import Jwt, JwtKeySet

func main() {
    keys: JwtKeySet = JwtKeySet()
    keys.addHS("current", "current-secret")
    keys.addHS("previous", "previous-secret")

    claims: dict[str, any] = {}
    claims["sub"] = "alice"
    token: str = Jwt.signHS256Kid(claims, "current", "current-secret", 300)

    print(keys.verifyHS256(token))
    keys.remove("previous")
}
```

Use `size()` and `kids()` to inspect a key set. `decodeHS256` and `decodeRS256`
return verified claims, while their `verify*` counterparts return only a
boolean.

## Security notes

- Never trust `Jwt.parseUnverified` output. It intentionally skips signature and
  claim verification and is only suitable for diagnostics or routing.
- `Jwt.extractKid` is also unverified. Treat the result only as a key-selection
  hint; verification must still succeed.
- Prefer `decode*` or `tryDecode*` when handling untrusted tokens.
- Use strong HMAC secrets, protect private keys, keep token lifetimes short, and
  remove retired keys only after the intended rotation window.

## Testing and formatting

```bash
python3 tests/run_lamjwt_tests.py --verbose
lamc fmt .
```

The test runner requires `lamc >= 1.16.0`, stages this checkout as
`extlibs/@lam/jwt` in an isolated temporary tree, compiles both migrated JWT test
programs without compiler caches, and checks their exact output.

## License

Licensed under the Apache License, Version 2.0. See [LICENSE](LICENSE).
Release history is recorded in [CHANGELOG.md](CHANGELOG.md).
