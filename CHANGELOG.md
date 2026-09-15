# Changelog

All notable changes to `@lam/jwt` are documented in this file.

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
