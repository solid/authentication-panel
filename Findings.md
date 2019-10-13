# Solid Authentication Panel Findings

## OAuth Implicit Flow

[Panel issue #14](https://github.com/solid/authentication-panel/issues/14)

Clients and Authorizations Servers *MUST* implement Authorization Code Grant [RFC6749] with PKCE [RFC7636]. Authorizations Servers *MAY* choose not to support Implicit Flow [RFC6749] due various security threats [oauth-browser-based-apps] and Clients *MUST NOT* rely on Authorization Servers having Implicit Flow available.

## References

*   [RFC7636]  Sakimura, N., Ed., Bradley, J., and N. Agarwal, "Proof Key for Code Exchange by OAuth Public Clients", [RFC 7636](https://tools.ietf.org/html/rfc7636), DOI 10.17487/RFC7636, September 2015
*   [RFC6749]  Hardt, D., Ed., "The OAuth 2.0 Authorization Framework", [RFC 6749](https://tools.ietf.org/html/rfc6749), DOI 10.17487/RFC6749, October 2012
*   [oauth-browser-based-apps]  Parecki, A and Waite, D. "OAuth 2.0 for Browser-Based Apps", [oauth-browser-based-apps](https://tools.ietf.org/html/draft-ietf-oauth-browser-based-apps-04)