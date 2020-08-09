# Solid-OIDC Authentication Spec - Draft

# Abstract

A key challenge on the path toward re-decentralizing user data on the Web is
the need to access multiple potentially untrusted resource servers securely.
This document aims to address that challenge by building on top of current Web
standards, to allow entities to authenticate in the Solid ecosystem.

# Status of This Document

This section describes the status of this document at the time of its publication. Other documents
may supersede this document. A list of current W3C publications and the latest revision of this
technical report can be found in the [W3C technical reports index](https://www.w3.org/TR/) at
https://www.w3.org/TR/.

This document is produced from work by
the [Solid Community Group](https://www.w3.org/community/solid/). It is a draft document that may,
or may not, be officially published. It may be updated, replaced, or obsoleted by other documents at
any time. It is inappropriate to cite this document as anything other than work in progress. The
source code for this document is available at the following
URI: <https://github.com/solid/authentication-panel>

This document was published by the
[Solid Authentication Panel](https://github.com/solid/process/blob/master/panels.md#authentication)
as a First Draft.

[GitHub Issues](https://github.com/solid/authentication-panel/issues) are preferred for discussion
of this specification. Alternatively, you can send comments to our mailing list. Please send them to
[public-solid@w3.org](mailto:public-solid@w3.org)
([archives](https://lists.w3.org/Archives/Public/public-solid/))

# Introduction

_This section is non-normative_

The [Solid project](https://solidproject.org/) aims to change the way Web
applications work today to improve privacy and user control of personal data
by using current standards, protocols, and tools, to facilitate building
extensible and modular decentralized applications based on [Linked
Data](https://www.w3.org/DesignIssues/LinkedData) design principles.

In order to allow entities to authenticate in the [Solid
ecosystem](https://solid.github.io/specification/), this specification
interoperably combines functionality based on widely implemented standards
such as [OAuth 2.0](https://tools.ietf.org/html/rfc6749) and [OpenID Connect
Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html).

In a changing world where privacy and control of digital identities are
becoming more pressing concerns, this specification addresses additional
functionality:

1. Ephemeral clients as a common use case.
2. Resource servers with no existing trust relationship with identity providers.

This specification is for:

* Authorization and Resource Server owners who want to enable applications to
obtain access to an HTTP service;

* Application developers who want to implement a client to obtain access to an
HTTP service.

## Out of Scope

_This section is non-normative_

At the time of writing, there is no demonstrated use case for a strongly asserted identity, however,
it is likely that authorization requirements will necessitate it.

# Terminology

_This section is non-normative_

This specification uses the terms "access token", "authorization server",
"resource server" (RS), "authorization endpoint", "token endpoint", "grant
type", "access token request", "access token response", and "client" defined
by The OAuth 2.0 Authorization Framework
\[[RFC6749](https://tools.ietf.org/html/rfc6749)\].

This specification uses the notion of Identity Provider (IdP) from the [Open
ID Connect Core 1.0
specification](https://openid.net/specs/openid-connect-core-1_0.html) (OIDC).
It should be noted that [The OAuth 2.0 Authorization
Framework](https://tools.ietf.org/html/rfc6749) (OAuth) refers to this same
entity as an Authorization Server.

This specification also uses the following terms:

**WebID** _as defined in [WebID 1.0](https://www.w3.org/2005/Incubator/webid/spec/identity/)_

A WebID is a URI with an HTTP or HTTPS scheme which denotes an Agent (Person, Organization, Group,
Device, etc.)

**JSON Web Token (JWT)** _as defined by [RFC7519](https://tools.ietf.org/html/rfc7519)_

A string representing a set of claims as a JSON object that is encoded in a JWS or JWE, enabling the
claims to be digitally signed or MACed and/or encrypted.

**JSON Web Key (JWK)** _as defined by [RFC7517](https://tools.ietf.org/html/rfc7517)_

A JSON object that represents a cryptographic key. The members of the object represent properties of
the key, including its value.

**Demonstration of Proof-of-Possession at the Application Layer (DPoP)** _as defined in the
[DPoP Internet-Draft](https://tools.ietf.org/html/draft-fett-oauth-dpop-04)_

A mechanism for sender-constraining OAuth tokens via a proof-of-possession mechanism on the
application level.

**DPoP Proof** _as defined by
[DPoP Internet-Draft](https://tools.ietf.org/html/draft-fett-oauth-dpop-04)_

A DPoP proof is a JWT that is signed (using JWS) using a private key chosen by the client.

**Proof Key for Code Exchange (PKCE)** _as defined by
[RFC7636](https://tools.ietf.org/html/rfc7636)_

An extension to the Authorization Code flow which mitigates the risk of an authorization code
interception attack.

# Conformance

_This section is non-normative_

All authoring guidelines, diagrams, examples, and notes in this document are non-normative.
Everything else in this specification is normative unless explicitly expressed otherwise.

The key words MAY, MUST, MUST NOT, RECOMMENDED, SHOULD, SHOULD NOT, and REQUIRED in this document
are to be interpreted as described in [BCP 14](https://tools.ietf.org/html/bcp14)
\[[RFC2119](https://www.w3.org/TR/2014/REC-cors-20140116/#refsRFC2119)\]
\[[RFC8174](https://www.w3.org/TR/2019/REC-vc-data-model-20191119/#bib-rfc8174)\] when, and only
when, they appear in all capitals, as shown here.

# Core Concepts

_This section is non-normative_

In the Solid ecosystem, an IdP may be controlled, supplied and used by any
entity, and may exist for any length of time.

This specification makes use of OAuth and OIDC best practices and assumes the
[Authorization](https://openid.net/specs/openid-connect-core-1_0.html#CodeFlowSteps)
Code Flow with PKCE, as per OIDC definition. It is also assumed that there is
no preexisting trust relationship with the IdP. This means dynamic, and static
client registration is entirely optional.

## WebID

_This section is non-normative_

A [WebID](https://www.w3.org/2005/Incubator/webid/spec/identity/) is a HTTP
URI denoting an agent, for example a person, organisation, or software. When a
WebID is dereferenced, server provides a representation of the WebID Profile
in an [RDF document](https://www.w3.org/TR/rdf11-concepts/#dfn-rdf-document)
which uniquely describes an agent denoted by a WebID. The WebID Profile can be
used by controlling agents to link with others to grant access to identity
resources as they see fit. WebIDs are an underpinning component in the Solid
ecosystem and are used as the primary identifier for users and client
applications in this specification.

# Basic Flow

_This section is non-normative_

> TODO: Add diagram.

The basic authentication and authorization flow is as follows:

1. The client requests a non-public resource from the RS.
2. The RS returns a 401 with a `WWW-Authenticate` HTTP header containing parameters that inform the
   client that a DPoP-bound Access Token is required.
3. The client presents its WebID to the IdP and requests an Authorization Code.
4. The client presents the Authorization Code and a DPoP proof, to the token endpoint.
5. The Token Endpoint returns a DPoP-bound Access Token and OIDC ID Token, to the client.
6. The client presents the DPoP-bound Access Token and DPoP proof, to the RS.
7. The RS validates the Access Token and DPoP header, then returns the requested resource.

# Proof of Identity

Client registration (static or dynamic) is not required in Solid-OIDC, thus Access Tokens need to be
bound to the client, to prevent token replay attacks. This is achieved by using a
[Distributed Proof-of-Possession](https://tools.ietf.org/html/draft-fett-oauth-dpop-04) (DPoP)
mechanism at the application layer.

# Token Instantiation

Assuming the token request and DPoP Proof are valid, the client MUST receive two tokens from the
IdP:

1. A DPoP-bound Access Token
2. An OIDC ID Token

These tokens require additional and/or modified claims for them to be compliant with the
authorization flow laid out in this document.

## DPoP-bound Access Token

The client MUST send the IdP a DPoP proof that is valid according to the
[DPoP Internet-Draft](https://tools.ietf.org/html/draft-fett-oauth-dpop-04).

The audience (`aud`) claim is required in OAuth, however, the DPoP token provides the full URL of
the request, making the `aud` claim redundant, so in Solid-OIDC the `aud` claim MUST be a string
with the value of `solid`.

An example Access Token:

```js
{
    "sub": "https://janedoe.com/web#id", // WebID of User
    "iss": "https://idp.example.com",
    "aud": "solid",
    "iat": 1541493724,
    "exp": 1573029723, // Identity credential expiration (separate from the ID token expiration)
    "cnf":{
      "jkt":"0ZcOCORZNYy-DWpqq30jZyJGHTN0d2HglBV3uiguA4I" // DPoP public key confirmation claim
    }
}
```

## OICD ID Token

The subject (`sub`) claim in the returning ID Token MUST be set to the user's WebID.

Example:

```js
{
    "iss": "https://idp.example.com",
    "sub": "https://janedoe.com/web#id",
    "aud": "https://client.example.com",
    "nonce": "n-0S6_WzA2Mj",
    "exp": 1311281970,
    "iat": 1311280970
}
```

# Resource Access

Ephemeral clients MUST use DPoP-bound Access Tokens, however, the RS MAY allow registered clients to
access resources using a traditional `Bearer` tokens.

## DPoP Validation

If a `cnf` claim is present in the DPoP-bound Access Token, then the DPoP
Proof MUST be present and validated using the methods outlined in the [DPoP
Internet-Draft](https://tools.ietf.org/html/draft-fett-oauth-dpop-04#section-4.2).

As defined, this includes ensuring that the DPoP Proof has not expired, and both the URL and the
HTTP method match that of the requested resource. If any of these checks fail, the RS MUST deny the
resource request.

## Validating the DPoP-bound Access Token

The public key in the fingerprint of the DPoP-bound Access Token MUST be
checked against the DPoP fingerprint to ensure a match, as outlined in the
[DPoP
Internet-Draft](https://tools.ietf.org/html/draft-fett-oauth-dpop-04#section-6).

### WebID Claim and Check

In order to prevent a malicious identity provider from issuing valid
DPoP-bound Access Tokens for arbitrary WebIDs, the following check is
required:

The `sub` claim of the DPoP-bound Access Token MUST be a WebID. This needs to
be dereferenced and checked against the `iss` claim in the DPoP-bound Access
Token. If the `iss` claim is different from the domain of the WebID, then the
RS MUST check the WebID document for a `solid:oidcIssuer` property to check
the token issuer is listed.

# Security Considerations

_This section is non-normative_

As this specification builds upon existing Web standards, security
considerations from OAuth, OIDC, PKCE, and the DPoP specifications may also
apply unless otherwise indicated. The following considerations should be
reviewed by implementors and system architects of this specification.

Some of the normative references with this specification point to documents
with a Living Standard or Draft status, meaning their contents can still
change over time. It is advised to monitor these documents, as such changes
might have security implications.

In addition to above considerations, implementors should consider the Security
Considerations in context of the [Solid
Ecosystem](https://solid.github.io/specification/).

## TLS Requirements

All TLS requirements outlined in
[OIDC Section 16.17](https://openid.net/specs/openid-connect-core-1_0.html#Security) apply to this
specification.

All tokens, client, and user credentials MUST only be transmitted over TLS.

## Client IDs

Implementors SHOULD expire client IDs that are kept in server storage to mitigate the potential for
a bad actor to fill server storage with unexpired or otherwise useless client IDs.

## Client Secrets

Client secrets SHOULD NOT be stored in browser local storage. Doing so will increase the risk of
data leaks should an attacker gain access to client credentials.

## Client Trust

_This section is non-normative_

Clients are ephemeral, client registration is optional, and most clients cannot keep secrets. These,
among other factors, are what makes client trust challenging.

# Privacy Considerations

_This section is non-normative_

## DPoP-bound Access Token Reuse

With JWTs being extendable by design, there is potential for a privacy breach
if DPoP-bound Access Tokens get reused across multiple resource servers. It is
not unimaginable that a custom claim is added to the DPoP-bound Access Token
on instantiation. This addition may unintentionally give other resource
servers consuming the DPoP-bound Access Token information about the user that
they may not wish to share outside of the intended RS.

# Acknowledgements

> TODO:

# References

> TODO:
