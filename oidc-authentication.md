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

This specification uses the terms "Credential", "Verifiable Credential",
"Issuer" and "Presentation" defined by Verifiable Credentials Data Model 1.0
[VC], the terms "Access Token", "Authorization Code", "Authorization Endpoint",
"Authorization Grant", "Authorization Server", "Client Authentication", "Client
Identifier", "Client Secret", "Grant Type", "ID Token", "Implicit Flow",
"Protected Resource", "Redirection URI", "Refresh Token", "Resource Owner",
"Resource Server", "Response Type", and "Token Endpoint" defined by OAuth 2.0
[RFC6749], the terms "Claim Name", "Claim Value", "JSON Web Token (JWT)", "JWT
Claims Set", and "Nested JWT" defined by JSON Web Token (JWT) [JWT], the terms
"Header Parameter" and "JOSE Header" defined by JSON Web Signature (JWS) [JWS],
the term "User Agent" defined by RFC 2616 [RFC2616], and the terms
"Authentication", "Authentication Request", "Authorization Code Flow", "Claim",
"End-User", "Entity", "Identifier", "Identity", "Implicit Flow", "OpenID
Provider (OP)", "Relying Party (RP)", "Validation" and "Verification" defined by
[OpenID Connect Core
1.0](https://openid.net/specs/openid-connect-core-1_0.html#Terminology) [OIDC].

##### entity
As defined by [OIDC]:

> Something that has a separate and distinct existence and that can be
> identified in a context. An End-User is one example of an Entity.

##### claim
As defined by [OIDC]:

> Piece of information asserted about an Entity.

##### identifier
As defined by [OIDC]:

> Value that uniquely characterizes an Entity in a specific context.

A [Web ID] or a [DID] is an example of a globally unique, non-pairwise
pseudonymous identifier.

##### controller
A person ([end-user]), group or organization that is responsible for an
[entity], and controls that entity's [identifier].

##### identifier document
A machine-readable document, obtainable by resolving a Web ID or DID
[identifier], that contains statements and metadata about the Entity being
identified. Typically, it contains cryptographic materials such
as public keys, links to service endpoints and authorized [OpenID Connect
Providers] (OPs), and other attributes that are helpful for purposes of
[authentication].

A Web ID Profile, and a DID Document, are both examples of identifier documents.

##### end-user
As defined by [OIDC]:

> Human participant.

##### identity
As defined by [OIDC]:

> Set of attributes related to an entity.

Here, identity specifically refers to a combination of an [identifier], and a
corresponding [identifier document] that is obtainable by resolving that
identifier.

##### authentication
As defined by [OIDC]:

> Process used to achieve sufficient confidence in the binding between the
> entity and the presented identity.

##### authentication flow
[OpenID Connect] built on top of OAuth 2's Authorization Grants, and
introduced the concept of Authentication Flow - a process where a [client] can
use a valid grant (such as the Implicit Grant, Authorization Code Grant, or
others) to perform [authentication].

The end result of any successful authentication flow is that the client
performing it receives one or more credentials from the [identity provider]. In
the case of classic OpenID Connect flows, those credentials are typically either
the [id token], an [access token] specific to a particular resource server, or
both.

In addition, OpenID Connect provides a way for a [relying party] to request
arbitrary [claims] or [credentials] inside the ID Token. In particular, this
specification uses that mechanism to allow a  [relying party] to request one or
more reusable credentials that enable audience-constrained [protected resource
requests] to multiple unaffiliated [resource servers] (see section [Multi
Resource Server (Multi-RS) Use Case] for more details).

##### Web ID
A globally unique [identifier] (typically an HTTPS URL) for an [entity], that
can be resolved to a Web ID Profile document.

##### user-agent
As defined by RFC 2616 [RFC2616]. Often, a formal name for a web browser. Note
that this is often separate from a [client] application (in many cases, client
apps written in Javascript run inside the browser).

##### resource controller
An [entity] with administrative permissions to a resource (e.g. `acl:Control`
access mode in the case of [WAC]). Alternatively, the [entity] in control of the
[resource server] (such as a [Solid pod] or storage space) in which the resource
resides.

Similar to the OAuth 2 and
[UMA 2](https://docs.kantarainitiative.org/uma/wg/rec-oauth-uma-grant-2.0.html)
concept of "resource owner", the resource controller MAY be an [end-user]
(natural person) or MAY be a non-human entity treated as a person for limited
legal purposes (legal person), such as a corporation.

Note the use of "controller" as opposed to "owner"; this specification (and
Solid in general), has no notion of owner.

A given resource can have zero, one, or multiple resource controllers.

##### protected resource
A resource to which access is constrained by an appropriate authorization
mechanism (such as Web Access Control [WAC] rules or Authorization Capabilities
[zCAP]).

##### client
An application, service, bot, or other software entity interacting with a
protected resource hosted by a [resource server] on behalf of a [requesting
party]. The term "client" does not imply any particular implementation
characteristics (such as whether the application executes on a server, a
desktop, inside a web browser, or on other devices).

The client is an [entity] in its own right, possessing an [identifier], and in
some cases, a corresponding [identifier document] containing client metadata.

##### requesting party
As defined by [UMA2]:

An [entity] (natural or legal person) that uses a [client] to seek access to a
[protected resource]. The requesting party may or may not be the same party as
the [resource controller].

##### Solid storage
A server compliant with the [Solid Specification] that acts as a [resource
server] (RS).

##### resource server (RS)
As defined by [OAuth2]:

> The server hosting the protected resources, capable of accepting and
> responding to protected resource requests

##### protected resource request
A request by a [client] to a [protected resource], carrying [authentication] or
authorization [credentials] such as an OAuth 2 Access Token, a Verifiable
Credential, or Proof of Possession token.

##### relying party (RP)
As defined by [OIDC]:

> OAuth 2.0 Client application requiring End-User Authentication and Claims
> from an OpenID Provider.

The [client] needs [authentication] related [claims]/[credentials] for UI
reasons (to display a user name and icon to indicate that they’re logged in, for
example), but also in order to present those credentials to the [resource
server] when making a [protected resource request]).

##### OpenID Provider (OP)
Also known as an Identity Provider (IdP) or the Authorization Server (AS).

As defined by [OIDC]:

> OAuth 2.0 Authorization Server that is capable of Authenticating the End-User
> and providing Claims to a Relying Party about the Authentication event and
> the End-User.

## 1.8 Trust Model

## 1.9 Overview

# 2. Concepts

## 2.1 Relationship between Authentication and Authorization

## 2.2 Identity

## 2.3 User Authentication

## 2.4 Client Authentication

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
// This ID Credential is signed by the IDP, NOT by the client
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
// This token is signed by the client
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
