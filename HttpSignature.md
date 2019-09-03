# HTTP-Signature Authentication for SoLiD

[HTTP-Signature](https://datatracker.ietf.org/doc/draft-cavage-http-signatures/) is an IETF RFC Draft that has been developing since 2013 for signing and authenticating HTTP messages. It has the advantage of being very simple and being  specified directly at the HTTP layer. As a result there are a [large number of implementations](https://github.com/w3c-dvcg/http-signatures/issues/1), and is already widely used (todo: link).  

The protocol allows a client to authenticate by signing a number of HTTP headers with it's private key. In order for the server to be able to verify this signature it needs to know the matching public key. This information must be transmitted by the client in the form of an opaque string known as a `keyId` (see [§2.1.1 keyId](https://tools.ietf.org/html/draft-cavage-http-signatures-11#section-2.1.1)). This string must enable the server to look up the key. How this is done is not specified by the protocol.

The protocol allows the key to be interpreted as a URL, and so the proposal here is to use an `https` URL identifier for the `keyId`. In order for a server to discover the key, it can simply dereference the document named by that key. 

### Sequence Diagram

The addition to the HTTP Signature protocol made here can be illustrated 
by the following Sequence Diagram:

```text
Client                          KeyID                            Resource
App                          Document                            Server
|                                |                                   |  
|-request URL ------------------------------------------------------>| 
|<-------------------------------------- 401 + WWW-Auth Sig header---|
|                                |                                   |
|-- sign headers+keyId---------------------------------------------->|
|                                |                       initial auth|
|                                |                       verification| 
|                                |                                   |
|                                |<----------------------GET keyId---| 
|                                |-return keyId doc----------------->|
|                                                        -verify sig |
|                                                        -verify wACL|
|                                                                    |
|<-------------------------------------------------answer resource---|
```                                                                   

The addition to the HTTP signature protocol is the request by the
resource server for the keyId document. 

Note: If the key document is local to the Resource server, the 
protocol indistinguishable in terms of requests to the HTTP-Signature 
document.

### The KeyId Document

The `keyId` is a Hash URL that refers to a key.
An example key would be: 
  `https://bob.example/keys/2019-09-02#k1` 
The URL without the hash refers to the `keyId` document,
which can be dereferences, so in the above case it would be
  `https://bob.example/keys/2019-09-02`

The KeyId document must contain a description of the public key in
RDF. If we were to use [the cert ontology](https://www.w3.org/ns/auth/cert#) used by WebID-TLS then this document would need to contain triples such as

```Turtle 
@prefix cert: <http://www.w3.org/ns/auth/cert#> .

<#k1>  a cert:RSAPublicKey;
     cert:modulus "00cb24ed85d64d794b..."^^xsd:hexBinary;
     cert:exponent 65537 .
```

(Manu Sporny's team developed another key ontology, which may be more
up to date. Find Link.)

### The Corresponding WebACL

The simplest WebACL for allowing a key would look as follows

```Turtle           
@prefix  acl:  <http://www.w3.org/ns/auth/acl#>.
@prefix cert: <http://www.w3.org/ns/auth/cert#> .

<#authorization1>
    a             acl:Authorization;
    acl:accessTo  <https://alice.example/docs/shared-file1>;
    acl:mode      acl:Read,
                  acl:Write, 
                  acl:Control;
    acl:agent    [ cert:key <https://bob.example/keys/2019-09-02#k1> ].
```

### Linking a WebKey to a WebID

This can be developed but goes beyond authentication.
