# Changelog

All notable changes to `@lam/jwt` are documented in this file.

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
