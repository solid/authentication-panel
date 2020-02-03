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

## 8.1 Use of `Authorization` header

## 8.2 Resource Server Credential Validation

# 9. Errors

# 10. Security Considerations

# 11. Privacy Considerations

# 12. Extensibility

# 13. Appendix

## 13.1 Relationship to Other Specifications
