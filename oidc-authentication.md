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

[RFC 7235]() defined the protocol for using the WWW-Authenticate HTTP response
header as a way for a Resource Server to issue one or more challenges to a
client, that is, to communicate which Authentication schemes it supports.

Solid uses this mechanism to allow a [Resource Server]() to communicate to a
[Client/Presenter]() which Authentication methods (and versions) it supports.

### Proposed WebID-OIDC 1.0 response example:

Example:

```
> GET /example/resource

< HTTP/1.1 401 Unauthorized
< WWW-Authenticate: DPoP realm=”https://example.com” scope=”openid webid”
```

* Realm param is optional
* Scope -- “`openid webid`” params a MUST.

Note: the DPoP authentication scheme is required by [DPoP draft spec](https://tools.ietf.org/html/draft-fett-oauth-dpop)

If RS supports both authentication schemes (Bearer and DPoP), multiple WWW-Authenticate headers may be returned.

```
WWW-Authenticate: DPoP realm=”https://example.com” scope=”openid webid”, Bearer realm=”https://example.com” scope=”openid webid”
```

# 4. Identity Provider Selection

# 5. Client Registration / Application Metadata

# 6. Authentication Flows

## 6.1 OIDC Authorization Code Flow

## 6.2 Constrained Device Flow

## 6.3 Implicit (Legacy) Flow

## 6.4 Multi Resource Server (Multi-RS) Use Case

## 6.5 Role of Local Authentication in OIDC Flows

# 7. Tokens and Credentials

# 8. Authenticated Requests

When making authenticated requests to [Resource Server]()s, a [Client]() uses
a combination of the `Authorization` header and an appropriate Proof of
Possession header (such as DPoP).

## 8.1 Use of Authorization header

### For Multi-RS / “Social App” Use Cases

For Multi-RS use cases, a client MUST use an `Authorization` header as well
as the `DPoP` proof-of-possession header.

**Authorization:** `DPoP <include JWT-encoded ID Credential here>`<br/>
**DPoP:** `<include a sender-constrained DPoP Proof JWT (see example below)>`

Example authenticated request:

```
GET /protectedresource HTTP/1.1
 Host: resource.example.org
 Authorization: DPoP eyJhbGciOiJFUzI1NiIsImtpZCI6IkJlQUxrYiJ9.eyJzdWI
  iOiJzb21lb25lQGV4YW1wbGUuY29tIiwiaXNzIjoiaHR0cHM6Ly9zZXJ2ZXIuZXhhbX
  BsZS5jb20iLCJhdWQiOiJodHRwczovL3Jlc291cmNlLmV4YW1wbGUub3JnIiwibmJmI
  joxNTYyMjYyNjExLCJleHAiOjE1NjIyNjYyMTYsImNuZiI6eyJqa3QiOiIwWmNPQ09S
  Wk5ZeS1EV3BxcTMwalp5SkdIVE4wZDJIZ2xCVjN1aWd1QTRJIn19.vsFiVqHCyIkBYu
  50c69bmPJsj8qYlsXfuC6nZcLl8YYRNOhqMuRXu6oSZHe2dGZY0ODNaGg1cg-kVigzY
  hF1MQ
 DPoP: eyJ0eXAiOiJkcG9wK2p3dCIsImFsZyI6IkVTMjU2IiwiandrIjp7Imt0eSI6Ik
  VDIiwieCI6Imw4dEZyaHgtMzR0VjNoUklDUkRZOXpDa0RscEJoRjQyVVFVZldWQVdCR
  nMiLCJ5IjoiOVZFNGpmX09rX282NHpiVFRsY3VOSmFqSG10NnY5VERWclUwQ2R2R1JE
  QSIsImNydiI6IlAtMjU2In19.eyJqdGkiOiJlMWozVl9iS2ljOC1MQUVCIiwiaHRtIj
  oiR0VUIiwiaHR1IjoiaHR0cHM6Ly9yZXNvdXJjZS5leGFtcGxlLm9yZy9wcm90ZWN0Z
  WRyZXNvdXJjZSIsImlhdCI6MTU2MjI2MjYxOH0.lNhmpAX1WwmpBvwhok4E74kWCiGB
  NdavjLAeevGy32H3dbF0Jbri69Nm2ukkwb-uyUI4AUg1JSskfWIyo4UCbQ
```

Example contents of `Authorization` header (this is the ID Credential in its
entirety, the contents of the `id_vc` claim from the IDP's ID Token):

```
{
  "sub": "https://janedoe.com/web#id",    <- Web ID / DID
  "iss": "https://idp.example.com",
  "aud": "https://client.example.com", // audience constrained to the client / presenter
  "iat": 1541493724,
  "exp": 1573029723,  <- identity credential expiration 
  "cnf":{
      // DPoP public key confirmation, per DPoP spec section 7
      "jkt":"0ZcOCORZNYy-DWpqq30jZyJGHTN0d2HglBV3uiguA4I"  
  }
}
```

Example DPoP Proof JWT (minted by Client app for each resource) (the
contents of the `DPoP` header):

```js
  {
     "typ":"dpop+jwt",
     "alg":"ES256",
     "jwk": {   // <- gets checked against the cnf claim in the ID Credential (from the Authorization header)
       "kty":"EC",
       "x":"l8tFrhx-34tV3hRICRDY9zCkDlpBhF42UQUfWVAWBFs",
       "y":"9VE4jf_Ok_o64zbTTlcuNJajHmt6v9TDVrU0CdvGRDA",
       "crv":"P-256"
     }
   }.{
     "jti":"-BwC3ESc6acc2lTc",
     "iat":1562262616,
      // the following fields are required only for the authenticated resource request (not for the token endpoint)
     "htm":"POST",
     "htu":"https://rs.example.com/protected/resource",   // <- audience-restriction mechanism
   }
```

### For “traditional” / Single RS Use Cases

For authenticated requests clients MUST do one of:

a) Either use DPoP-authenticated requests as per Multi-RS section above, or<br/>
b) Use “traditional” style access token (and derive WebID url from it as per
   this spec)

```
  GET /protectedresource HTTP/1.1
   Host: resource.example.org
   Authorization: Bearer eyJhbGciOi...kVigzY
```

Note the use of `Bearer` authentication scheme, as opposed to the `DPoP` scheme
from previous section.

```
{
  "iss": "http://idp.example.com",
  "sub": "https://janedoe.com/web#id",
  "aud": "https://rs.example.com",
  "exp": 1311281970,
  "iat": 1311280970,
   ...
}
```

## 8.2 Resource Server Credential Validation

A [Resource Server]() must validate all authenticated requests using the
required validation steps for each method, plus the (Solid-specific)
[Authorized WebID Provider
Confirmation](https://github.com/solid/webid-oidc-spec#webid-provider-confirmation)
step.

### Bearer Tokens (Authorization: Bearer), Single RS use case

1. Validate JWS token as per the JOSE rules (expiration, 'not-before’
   timestamp, signature validation by the issuer, audience claim matches the RS)
2. (Solid-Specific) Validate that the issuer is an authorized OIDC issuer by
   fetching the WebID Profile, and performing the Authorized WebID Provider
   Confirmation step.

### DPoP Tokens (Authorization: DPoP), Multi RS use case

1. Validate the Proof token from the DPoP: header as per
   https://tools.ietf.org/html/draft-fett-oauth-dpop-03#section-4.2
   (Checking DPoP Proofs)
2. Validate the ID Credential from the `Authorization: DPoP` header, as per
   JOSE rules, plus
   https://tools.ietf.org/html/draft-fett-oauth-dpop-03#section-7
   (Public Key Confirmation) section.
3. (Solid-Specific) Validate that the issuer is an authorized OIDC issuer by
   fetching the WebID Profile, and performing the Authorized WebID Provider
   Confirmation step.

# 9. Errors

# 10. Security Considerations

# 11. Privacy Considerations

# 12. Extensibility

# 13. Appendix

## 13.1 Relationship to Other Specifications
