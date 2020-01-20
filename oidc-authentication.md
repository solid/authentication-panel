# WebID-OIDC Authentication Spec v1.0

# Table of Contents

* 1 - Introduction
    - Audience
    - Use Cases
    - Design Goals and Rationale
    - Compatibility with OpenID Connect
    - Difference from OpenID Connect
    - Difference from previous spec version (WebID-OIDC v0.1)
    - Out of Scope
    - Terminology
    - Trust Model
    - Overview
* 2 - Concepts
    - 2.1 Relationship between Authentication and Authorization
    - 2.2 Identity
    - 2.3 User Authentication
    - 2.4 Client (App/Service) Authentication
* 3 - Authentication Scheme Discovery
* 4 - Identity Provider Selection
    - Issuer Discovery from WebID Profile
* 5 - Client Registration / Application Metadata
* 6 - Authentication Flows
    - OIDC Authorization Code Flow
    - Self-Issued OpenID Connect (SIOP) Flow
    - Constrained Device Flow
    - Implicit (Legacy) Flow
    - Multi Resource Server (Multi-RS) Use Case
    - Role of Local Authentication in OIDC Flows
* 7 - Tokens and Credentials
* 8 - Authenticated Requests
    - Use of Authorization header
    - Resource Server Credential Validation
* 9 - Errors
* 10 - Security Considerations
* 11 - Privacy Considerations
* 12 - Extensibility
    - Adding Authentication schemes
    - Adding Proof of Possession Mechanisms
    - Interoperability with Existing Authentication schemes (SAML, etc)
    - Interoperability with DID Auth
    - Interoperability with Verifiable Credentials
* 13 - Appendix
    - Relationship to Other Specifications

# 1. Introduction

## 1.1 Audience

## 1.2 Use Cases

## 1.3 Design Goals and Rationale

## 1.4 Compatibility with OpenID Connect

## 1.5 Difference from OpenID Connect

## 1.6 Difference from previous spec version (WebID-OIDC v0.1)

## 1.6 Out of Scope

## 1.7 Terminology

## 1.8 Trust Model

## 1.9 Overview

# 2. Concepts

## 2.1 Relationship between Authentication and Authorization

## 2.2 Identity

## 2.3 User Authentication

## 2.4 Client (App/Service) Authentication

# 3. Authentication Scheme Discovery

# 4. Identity Provider Selection

# 5. Client Registration / Application Metadata

# 6. Authentication Flows

## 6.1 OIDC Authorization Code Flow

## 6.2 Constrained Device Flow

## 6.3 Implicit (Legacy) Flow

## 6.4 Multi Resource Server (Multi-RS) Use Case

## 6.5 Role of Local Authentication in OIDC Flows

# 7. Tokens and Credentials

### ID Tokens

The ID Token, defined in [OIDC Core 1.0](), has two primary purposes:

1. Authenticates the user to the RP / Client application (so that it can
   display their name / picture, for example)
2. Serve as a general-purpose container for arbitrary custom claims.

[OIDC Core 1.0 section 5]() defines the Claims mechanism, as a way to request
arbitrary claims and credentials inside the `id_token`. (Either via `scope`
param (section
https://openid.net/specs/openid-connect-core-1_0.html#ScopeClaims) or via the
`claims` request param (section
https://openid.net/specs/openid-connect-core-1_0.html#ClaimsParameter))

Example of a traditional ID Token:

```
{
  "iss": "http://server.example.com",
  "sub": "248289761001",
  "aud": "s6BhdRkqt3",
  "nonce": "n-0S6_WzA2Mj",
  "exp": 1311281970,
  "iat": 1311280970,
  "nickname": "JaneDoe",
  "picture": "http://example.com/janedoe/me.jpg",
  <any other custom claims>
}
```

Custom claims that we use currently: `webid`.
Custom claims this spec proposes: `id_vc`.

### Identity Credential

The Identity Credential is requested and received as a custom claim, inside
the existing OIDC ID Token:

```
{
  "iss": "https://idp.example.com",
  "sub": "https://janedoe.com/web#id",
  "aud": "https://client.example.com",
  "nonce": "n-0S6_WzA2Mj",
  "exp": 1311281970,
  "iat": 1311280970,
  “id_vc”: <reusable ID credential, in Verifiable Credentials format, encoded as a JWT>
}
```

- Is reused (with DPoP mechanism) for the Multi-RS use case
- For non-Multi-RS cases (and for compatibility with current IdPs and clients), use of ID Credential is optional; a WebID URL can be derived from plain ID Token or Access Token (see below).

Decoded `id_vc`, a re-usable identity credential, for use with DPoP headers:

```
{
  "sub": "https://janedoe.com/web#id",    <- Web ID / DID
  "iss": "https://idp.example.com",
  "aud": "https://client.example.com", // audience constrained to the client / presenter
  "iat": 1541493724,
  "exp": 1573029723,  <- identity credential expiration (separate from the ID token)
  "cnf":{
         // DPoP public key confirmation, per DPoP spec section 7
       "jkt":"0ZcOCORZNYy-DWpqq30jZyJGHTN0d2HglBV3uiguA4I"  
  }
}
```

# 8. Authenticated Requests

## 8.1 Use of `Authorization` header

## 8.2 Resource Server Credential Validation

# 9. Errors

# 10. Security Considerations

# 11. Privacy Considerations

# 12. Extensibility

# 13. Appendix

## 13.1 Relationship to Other Specifications
