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

This section is non-normative.

## 2.1 Purpose of Authentication

The Solid Project (and similar decentralized personal data servers) use
[authentication]() for the following purposes:

1. **To enable authorization**. First and foremost, authentication is used as a
  mechanism to enable rule-based access control (WAC) (although see Section
  2.1.1). Authorization requires knowing _both_ the identity of the user and the
  identity of the client, and as a result, authentication needs to provide
  (whenever technically possible) a tuple of: *(controller id, client id)*. To
  put it another way, for any given action (such as an [authenticated
  request]() to a data store), the access control mechanism needs to know _who_
  performed the action, and _what software/client_ they were using to perform
  it.

2. **To enable discovery**. Resolving a controller's or a client's identifier
  results in an [identifier document]() which is a useful starting point the
  discovery of user and client metadata. For example, a client application
  frequently requires some UI elements with which to address the user (a
  username or a profile picture), as well as the user's preferences and other
  data relevant to this app.

3. **For auditing**. Many use cases require the keeping of audit trails and
  access logs, which requires knowing who and what accessed a given resource or
  performed an action.

### 2.1.1 Authorization Without Authentication

Authorization systems sometimes require the ability to perform access control
decisions _without_ authentication, without identifying who is performing an
action, either because the protected resource is sufficiently low-value (or
low-threat if stolen), or due to practical requirements such as when sharing an
online document for dissemination and collaboration with an unknown audience.

As a result, most deployed authorization systems tend to employ a _hybrid_
paradigm: they allow both identity-based and capability-based access control.
Systems that prefer strict identity based Access Control Lists nevertheless
develop capability links (unguessable tokens or URLs that allow access to the
resource). Similarly, even systems such as [Authorization Capabilities]()
(originally known as Object Capabilities) that are staunchly _against_
authentication or using identifiers for access control, are forced to
accommodate authentication requirements (if only for purposes of auditing) --
see, for example, the [Horton protocol](http://www.erights.org/download/horton/document.pdf)
used by the object capabilities community.

Although authorization is largely out of scope for this document, it is worth
mentioning that this specification aims to support this hybrid approach, and
provide functionality for authentication as well as token-based capabilities.

## 2.2 Identity

For a given [entity]() (whether a person, an organization or a software agent),
the term [identity]() refers to a combination of: an [identifier]() (such as a
WebID or a DID), and a set of [claims]() (metadata and other attributes) that
typically reside in an [identifier document]() (such as a WebID Profile or a
DID Document).

In this specification, we are concerned primarily with two types of identities.

The first one is the identity of users (or [controller]()s), which are entities
responsible for an action performed on a protected resource.

The second one is the [client]() identity, since the WAC authorization system
often requires the knowledge of what software client was used to perform an
action, and restricts access accordingly.

Whenever possible, this specification prefers identifiers that are _resolvable_,
and result in identifier documents.

### 2.2.1 Controller Identifiers (WebID / DID)

- For people and organizations only. For application or service identity, see
  sections 2.2.3 and 2.2.4.
- Link to the WebID section of the spec.
- Globally unique, non-pairwise-pseudonymous.

### 2.2.2 Classifying Clients

There are many different kinds of software that we’re including in the broad
term [client]().

**We can classify clients by:**

- By deployment type: Server-side web apps, in-browser JS web apps, mobile
  apps, desktop apps, constrained apps and IoT devices.
- Clients that have their own identity (bots/services) vs clients that are
  always “piloted” by a user (apps)
- “Attended” vs “Unattended” clients
- By ability to keep secrets (Public vs Confidential vs Semi-Confidential
  (native))
- By connectivity (always-on vs intermittent, Internet-accessible vs firewalled/
  intranet-only, local/air-gapped, progressive/offline-capable).

It is important to distinguish between **strongly-identifiable** clients
and **weakly-identifiable** (or user-identifiable) clients.

**Strongly identifiable clients** can be identified by 3rd parties
independently from their user/controller. Only server-side web apps are
strongly identifiable -- as  confidential clients, they can keep secrets and
can present attestations and  third-party credentials via DNS / domain
certificates.

**Native apps** - strongly-identifiable in theory (since they are able to keep
secrets on an instance level), but not in practice (the OS manufacturers do not
make their trust infrastructure available).

**Weakly- or User-identifiable clients** - in-browser JS apps. Public clients,
not able to keep secrets on an instance level. Strongly authenticatable only to
their user/controller. Authenticatable to arbitrary third-party Resource
Servers only _as long as the app's controller cooperates and can be trusted_.

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

Client Registration for Solid clients is optional. The [Client]() can proceed
without registration as if it had registered with the [OP]() and obtained the
following [OAuth 2.0 Dynamic Client
Registration](https://tools.ietf.org/html/rfc7591) response:

```
client_id: <Client ID such as a client Web ID or redirect_uri>
client_secret_expires_at: 0
```

Otherwise, Static or Dynamic registration proceeds exactly as outlined in the
existing OIDC and OAuth 2 specifications.

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
