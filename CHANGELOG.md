# Changelog

All notable changes to `@lam/jwt` are documented in this file.

## [3.1.4] - 2026-11-14

### Fixed

- An empty HMAC secret is refused. `golang-jwt` signs and verifies with an
  empty key, so a service whose secret was missing (an unset environment
  variable) issued tokens anyone could forge, and accepted them:
  `signHS256` now returns a `JwtSignError`, `signHS512`/`signHS256Kid` return
  `""`, `verifyHS256`/`verifyHS512` return `false`, `decodeHS256` a
  `JwtVerifyError`, `decodeHS512` `{}`, and a `JwtKeySet` never verifies with
  an empty secret registered by `addHS`.

### Tests

- `tests/test_jwt_edge_cases.lam`: `iat`/`exp` stamping (TTL 0 and negative
  omit `exp`), non-ASCII claims and secrets, nested and 100 000-character
  claims, expired and not-yet-valid tokens, leeway (0, clamped negative),
  `decodeHS512`, HS256/HS512 not interchangeable, empty/malformed/tampered
  tokens with typed errors, RS256 bad PEM keys and kid, diagnostics on
  garbage.
- `tests/test_jwt_keyset_composed.lam`: typed `JwtKeySetError`s of
  `addRSA`/`addEC`/`addJwk`/`loadJwks`, an empty key set, shared kids across
  key families, the empty-secret regression, and a rotation stress (a
  `TokenAuthority` class, 60 async verifications while kids rotate, 20
  concurrent ES256 decodes, `?` and `do/catch`, comprehensions).

## [3.1.3] - 2026-11-03

### Changed

- Compatible with `lamc` 4.x: `compatibility.lamc = ">=1.18, <5"` (test runner too).

## [3.1.2] - 2026-10-20

### Changed

- Compatible with `lamc` 3.x: `compatibility.lamc = ">=1.18, <4"` (test runner too).

## [3.1.1] - 2026-10-18

### Changed

- Compatible with `lamc` 2.x: `compatibility.lamc = ">=1.18, <3"` (the test
  runner accepts the same range). Tested with `lamc` 2.0.0; no API change.

## [3.1.0] - 2026-10-21

### Added

- ECDSA: `Jwt.signES256`, `signES384`, `signES512` (PEM private keys,
  optional `kid`), `Jwt.decodeES` and `verifyES` (PEM public keys). The
  token's algorithm must match the key's curve (P-256/ES256, P-384/ES384,
  P-521/ES512).
- `JwtKeySet.addEC(kid, publicKeyPem)`; `addJwk` and `loadJwks` accept EC
  entries (`crv`, `x`, `y`; points off the curve are rejected) and skip
  `use: enc` entries.
- `JwtKeySet.decode(token)` / `verify(token)`: verify any asymmetric token,
  RS256/384/512 and PS256/384/512 against the RSA keys or ES256/384/512
  against the EC keys of the matching curve, `kid` first; HMAC, `none` and
  algorithm-confusion tokens are refused. Use it for OpenID Connect ID
  tokens.

### Testing

- `tests/test_jwt_ec.lam`: round trips per curve, wrong keys and curves,
  key-set selection with and without `kid`, PS256, refused HS256/`none`/
  confusion/expired tokens, mixed RSA+EC JWKS, invalid EC JWKs, rotation and
  HS/RS regressions.

## [3.0.0] - 2026-10-15

### Removed

- **Breaking:** removed the root `tag()` helper. The package name and version
  are declared in `lamlib.toml`, and the generic top-level function made this
  package impossible to link together with any other package (or application)
  that also declared `tag()`, because all Lam modules share one Go package.

## [2.0.0] - 2026-09-30

### Changed

- **Breaking:** Replaced the `trySignHS256`, `tryDecodeHS256`, and
  `tryDecodeRS256` twins with single unsuffixed typed `Result` APIs.
- **Breaking:** `JwtKeySet.decodeRS256` now returns
  `Result[dict[str, any]]`; its `tryDecodeRS256` twin was removed.
- **Breaking:** Renamed the `JwtKeySet` constructor from `__init__` to `init`.
- Added concrete payload types to all public `Result` signatures.
- Raised the minimum compatible Lammergeier compiler to 1.18 (`^1.18`) and
  updated package examples and tests for the new contracts.

### Testing

- Added installed-package composition coverage for context-driven encrypted JSON
  claims, HS256 key rotation, and typed `Result` propagation through `do/catch`.

## [1.0.0] - 2026-09-13

### Added

- Initial standalone `@lam/jwt` package, migrated from Lammergeier's JWT
  standard-library implementation.
- `Jwt` signing, verification, decoding, `Result`, leeway, diagnostic, and
  `kid` APIs for HS256, HS512, and RS256 tokens.
- Thread-safe `JwtKeySet` support for HMAC and RSA keys, JWKS loading, key
  selection, and rotation.
- Scoped package-root exports, pinned `github.com/golang-jwt/jwt/v5 v5.3.1`
  integration, package tests, documentation, and Apache-2.0 licensing.
