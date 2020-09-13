# Solid-OIDC Authentication Spec - Draft

# Abstract

A key challenge on the path toward re-decentralizing user data on the Worldwide Web is the need to
access multiple potentially untrusted resources servers securely. This document aims to address that
challenge by building on top of current and future web standards, to allow entities to authenticate
within a Solid ecosystem.

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

The [Solid project](https://solidproject.org/) aims to change the way web applications work today to
improve privacy and user control of personal data by utilizing current standards, protocols, and
tools, to facilitate building extensible and modular decentralized applications based on
[Linked Data](https://www.w3.org/standards/semanticweb/data) principles.

This specification is written for Authorization and Resource Server owners intending to implement
Solid-OIDC. It is also useful to Solid application developers charged with implementing a Solid-OIDC
client.

The [OAuth 2.0](https://tools.ietf.org/html/rfc6749) and
[OpenID Connect Core 1.0](https://openid.net/specs/openid-connect-core-1_0.html) web standards were
published in October 2012, and November 2014, respectively. Since publication, they have increased
with a marked pace and have claimed wide adoption with extensive _'real-world'_ data and experience.
The strengths of the protocols are now clear, however, in a changing eco-system where privacy and
control of digital identities are becoming more pressing concerns, it is also clear that additional
functionality is required.

The additional functionality is aimed at addressing:

1. Ephemeral Clients as a common use case.
2. Resource servers with no existing trust relationship with identity providers.

## Out of Scope

_This section is non-normative_

At the time of writing, there is no demonstrated use case for a strongly asserted identity, however,
it is likely that authorization requirements will necessitate it.

# Terminology

_This section is non-normative_

This specification uses the terms "access token", "authorization server", "resource server" (RS),
"authorization endpoint", "token endpoint", "grant type", "access token reques", "access token
response", and "client" defined by The OAuth 2.0 Authorization Framework
\[[RFC6749](https://tools.ietf.org/html/rfc6749)\].

Throughout this specification, we will use the term Identity Provider (IdP) in line with the
terminology used in the
[Open ID Connect Core 1.0 specification](https://openid.net/specs/openid-connect-core-1_0.html)
(OIDC). It should be noted that
[The OAuth 2.0 Authorization Framework](https://tools.ietf.org/html/rfc6749) (OAuth) refers to this
same entity as an Authorization Server.

This specification also uses the following terms:

**WebID** _as defined in the
[WebID 1.0 Editors Draft](https://dvcs.w3.org/hg/WebID/raw-file/tip/spec/identity-respec.html)_

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

**International Resource Identifier (IRI)** as defined by [RFC3987](https://tools.ietf.org/html/rfc3987)

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

In a decentralized ecosystem, such as Solid, an IdP may be a preexisting IdP or vendor, or at the
other end of the spectrum, a user-controlled IdP.

Therefore, this specification makes extensive use of OAuth and OIDC best practices and assumes the
[Authorization](https://openid.net/specs/openid-connect-core-1_0.html#CodeFlowSteps) Code Flow with
PKCE, as per OIDC definition. It is also assumed that there is no preexisting trust relationship
with the IdP. This means dynamic, and static client registration is entirely optional.

## WebIDs

_This section is non-normative_

In line with Linked Data principles, a
[WebID](https://dvcs.w3.org/hg/WebID/raw-file/tip/spec/identity-respec.html) is a HTTP URI that,
when dereferenced, resolves to a profile document that is structured data in an
[RDF format](https://www.w3.org/TR/2014/REC-rdf11-concepts-20140225/). This profile document allows
people to link with others to grant access to identity resources as they see fit. WebIDs underpin
Solid and are used as a primary identifier for Users and Client applications in this specification.

# Basic Flow

_This section is non-normative_

> TODO: Add diagram.

The basic authentication and authorization flow is as follows:

1. The Client requests a non-public resource from the RS.
2. The RS returns a 401 with a `WWW-Authenticate` HTTP header containing parameters that inform the
   Client that a DPoP-bound Access Token is required.
3. The Client presents its Client Identifier and the associated Secret to the IdP and requests an
   Authorization Code.
4. If granted, the Client presents the Authorization Code and a DPoP proof, to the Token Endpoint.
5. The Token Endpoint returns a DPoP-bound Access Token and OIDC ID Token, to the Client.
6. The Client presents the DPoP-bound Access Token and DPoP proof, to the RS.
7. The RS gets the public key from the IdP and uses it to validate the DPoP proof and Access Token.
8. If the DPoP proof and Access Token are valid then The RS returns the requested resource.

# Client Identifiers

OAuth and OIDC require the Client application to identify itself to the IdP and RS by presenting a
[client identifier](https://tools.ietf.org/html/rfc6749#section-2.2) (Client ID). Solid applications
SHOULD use a WebID as their Client ID.

## WebID

The WebID itself MUST resolve to an RDF document that MUST include a single `solid:oidcRegistration`
property. That property MUST be a JSON serialization of an OIDC client registration, per the definition
of client registration metadata from \[[RFC7591](https://tools.ietf.org/html/rfc7591#section-2)\].

Also, the IdP MUST dereference the Client's WebID document and match any Client-supplied parameters,
with the values in the Client's WebID document.

Further, the `redirect_uri` provided by the Client MUST be included in the registration `redirect_uris`
list.

NOTE: the method by which the IdP resolves the WebID to an RDF document, is defined in
\[[WebID 1.0](https://www.w3.org/2005/Incubator/webid/spec/identity/#processing-the-webid-profile)\].
This example uses [Turtle](https://www.w3.org/TR/turtle/):

```
https://app.example/webid#id


@prefix solid: <http://www.w3.org/ns/solid/terms#> .

<#id> solid:oidcRegistration """{
    "client_id" : "https://app.example/webid#id",
    "redirect_uris" : ["https://app.example/callback"],
    "client_name" : "Solid Application Name",
    "client_uri" : "https://app.example/",
    "logo_uri" : "https://app.example/logo.png",
    "tos_uri" : "https://app.example/tos.html",
    "scope" : "openid profile offline_access",
    "grant_types" : ["refresh_token","authorization_code"],
    "response_types" : ["code"],
    "default_max_age" : 60000,
    "require_auth_time" : true
    }""" .
```

## The Public WebID

Ephemeral Clients MAY use the identifier `http://www.w3.org/ns/solid/terms#PublicOidcClient`. If the
Client uses this identifier then IdP MAY accept any `redirect_uri` as valid. Since it is public, the
Client is effectively anonymous to the RS.

## OIDC Registration

If the Client does not use a WebID then it MUST use a client identifier registered with the IdP via
either OIDC dynamic or static registration.

# Token Instantiation

Assuming the Client ID and Secret, and the DPoP Proof are valid, the IdP MUST return two tokens to
the Client:

1. A DPoP-bound Access Token
2. An OIDC ID Token

## DPoP-bound Access Token

The DPoP-bound Access Token MUST be a valid JWT. See \[[RFC7519](https://tools.ietf.org/html/rfc7519)\].

When requesting an DPoP-bound Access Token, the Client MUST send the IdP a DPoP proof that is valid
according to the [DPoP Internet-Draft](https://tools.ietf.org/html/draft-fett-oauth-dpop-04).

The DPoP-bound Access Token MUST contain at least these claims:

`sub` — REQUIRED. This claim MUST be the user's WebID.

`iss` — REQUIRED. The issuer claim MUST be a valid URL of the IdP instantiating this token.

`aud` — REQUIRED. However, the DPoP token provides the full URL of the request, making the `aud`
claim redundant, so in Solid-OIDC the `aud` claim MUST be a string with the value of `solid`.

`iat` — REQUIRED. The issued-at claim is the time at which this crendential was issued.

`exp` — REQUIRED. This credential expiration is seperate from the ID tokens expiration.

`cnf` — For all flows that require DPoP, the confirmation claim is REQUIRED, as per
[DPoP Internet-Draft](https://tools.ietf.org/html/draft-fett-oauth-dpop-04#section-7) specification.

`client_id` — REQUIRED in all flows except OIDC registration.

An example DPoP-bound Access Token:

```js
{
    "sub": "https://janedoe.com/web#id",
    "iss": "https://idp.example.com",
    "aud": "solid",
    "iat": 1541493724,
    "exp": 1573029723,
    "cnf":{
      "jkt":"0ZcOCORZNYy-DWpqq30jZyJGHTN0d2HglBV3uiguA4I"
    },
    "client_id": "https://client.example.com/web#id"
}
```

## OIDC ID Token

The subject (`sub`) claim MUST be the user's WebID.

Example:

```js
{
    "iss": "https://idp.example.com",
    "sub": "https://janedoe.com/web#id",
    "aud": "https://client.example.com",
    "nonce": "n-0S6_WzA2Mj",
    "exp": 1311281970,
    "iat": 1311280970,
}
```

# Resource Access

## DPoP Proof Validation

A DPoP Proof that is valid according to
[DPoP Internet-Draft, Section 4.2](https://tools.ietf.org/html/draft-fett-oauth-dpop-04#section-4.2),
MUST be present when a DPoP-bound Access Token is used.

## Access Token Validation

The DPoP-bound Access Token MUST be validated according to
[DPoP Internet-Draft, Section 7](https://tools.ietf.org/html/draft-fett-oauth-dpop-04#section-7)
but the RS MAY perform additional verification in order to determine whether to grant access to the
resource.

The user's WebID in the `sub` claim MUST be dereferenced and checked against the `iss` claim in the
Access Token. If the `iss` claim is different from the domain of the WebID, then the RS MUST check
the WebID document for a `solid:oidcIssuer` property to check the token issuer is listed. This
prevents a malicious identity provider from issuing valid Access Tokens for arbitrary WebIDs.

# Security Considerations

_This section is non-normative_

As this specification builds upon existing web standards, security considerations from OAuth, OIDC,
PKCE, and the DPoP specifications may also apply unless otherwise indicated. The following
considerations should be reviewed by implementors and system/s architects of this specification.

## TLS Requirements

All TLS requirements outlined in \[[BCP195](https://tools.ietf.org/html/bcp195)\] apply to this
specification.

All tokens, Client, and User credentials MUST only be transmitted over TLS.

## Client IDs

An RS SHOULD assign a fixed set of low trust policies to any client identified as anonymous.

Implementors SHOULD expire Client IDs that are kept in server storage to mitigate the potential for
a bad actor to fill server storage with unexpired or otherwise useless Client IDs.

## Client Secrets

Client secrets SHOULD NOT be stored in browser local storage. Doing so will increase the risk of
data leaks should an attacker gain access to Client credentials.

## Client Trust

_This section is non-normative_

Clients are ephemeral, client registration is optional, and most Clients cannot keep secrets. These,
among other factors, are what makes Client trust challenging.

# Privacy Considerations

_This section is non-normative_

## Access Token Reuse

With JWTs being extendable by design, there is potential for a privacy breach if Access Tokens get
reused across multiple resource servers. It is not unimaginable that a custom claim is added to the
Access Token on instantiation. This addition may unintentionally give other resource servers
consuming the Access Token information about the user that they may not wish to share outside of the
intended RS.

# Acknowledgments

_This section is non-normative_

The Solid Community Group would like to thank the following individuals for reviewing and providing
feedback on the specification (in alphabetical order):

Tim Berners-Lee, Justin Bingham, Sarven Capadisli, Aaron Coburn, Matthias Evering, Jamie Fiedler,
Michiel de Jong, Ted Thibodeau Jr, Kjetil Kjernsmo, Pat McBennett, Adam Migus, Jackson Morgan, Davi
Ottenheimer, Justin Richer, severin-dsr, Henry Story, Michael Thornburgh, Emmet Townsend, Ruben
Verborgh, Ricky White, Paul Worrall, Dmitri Zagidulin.

# References

> TODO:
