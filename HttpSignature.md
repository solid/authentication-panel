# HTTP-Sig Authentication for SoLiD

[Signing HTTP Messages](https://datatracker.ietf.org/doc/draft-cavage-http-signatures/) (henceforth `HTTP-Sig`) is an IETF RFC Draft that has been developing since 2013 for signing and authenticating HTTP messages. It has the advantage of being very simple and being  specified directly at the HTTP layer. As a result there are a [large number of implementations](https://github.com/w3c-dvcg/http-signatures/issues/1), and is already widely used (todo: link).  

The protocol allows a client to authenticate by signing a number of HTTP headers with it's private key. In order for the server to be able to verify this signature it needs to know the matching public key. This information must be transmitted by the client in the form of an opaque string known as a `keyId` (see [§2.1.1 keyId](https://tools.ietf.org/html/draft-cavage-http-signatures-11#section-2.1.1)). This string must enable the server to look up the key. How this is done is not specified by the protocol.

The protocol allows the key to be interpreted as a URL, and so the proposal here is to use an `https` URL identifier ending with a fragment for the `keyId`. In order for a server to discover the key, it can simply fetch the `keyId Document`, whose URL is given by the `keyId` URL without the fragment identifier (see [§3 of RFC 3986: Uniform Resource Identifier (URI): Generic Syntax](https://tools.ietf.org/html/rfc3986?#section-3)).  

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
resource server for the `keyId document`. 

Note: If the key document is local to the Resource server, the 
protocol is indistinguishable in terms of requests to the current `HTTP-Sig` specification.

### The KeyId Document

The `keyId` is a  URL that refers to a key.
An example key would be: 
  `https://bob.example/keys/2019-09-02#k1` 
The URL without the hash refers to the `keyId` document,
which can be dereferenced, so in the above case it would be
  `https://bob.example/keys/2019-09-02`

The KeyId document must contain a description of the public key in
an RDF format. If we were to use [the cert ontology](https://www.w3.org/ns/auth/cert#) used by WebID-TLS then this document would need to contain triples such as

```Turtle 
@prefix cert: <http://www.w3.org/ns/auth/cert#> .

<#k1>  a cert:RSAPublicKey;
     cert:modulus "00cb24ed85d64d794b..."^^xsd:hexBinary;
     cert:exponent 65537 .
```

(Manu Sporny's team developed another key ontology, which may be more
up to date. Find Link.)

### The Corresponding WebACL

The WebACL for allowing an agent in possession of a specific key access, 
can simply refer to it.

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

Or instead of listing agents one by one one can use an agent group 
description:

```turtle
<#authorization1>
    a             acl:Authorization;
    acl:accessTo  <https://alice.example/docs/shared-file1>;
    acl:mode      acl:Read,
                  acl:Write, 
                  acl:Control;
    acl:agentGroup <https://alice.example.com/work-groups#Accounting>.   
```             

The agent Group document located at `https://alice.example.com/work-groups` can describe the users in various ways including by `keyId` as in this example 

```turtle
<#Accounting>
    a                vcard:Group;
    vcard:hasUID     <urn:uuid:8831CBAD-1111-2222-8563-F0F4787E5398:ABGroup>;
    dc:created       "2013-09-11T07:18:19+0000"^^xsd:dateTime;
    dc:modified      "2015-08-08T14:45:15+0000"^^xsd:dateTime;

    # Accounting group members:
    vcard:hasMember  [ cert:key <https://bob.example/keys/2019-09-02#k1> ],
                     [ cert:key <https://candice.example/clefs/doc1#clef> ].
```

We can think of current HTTP Signature implementations as using Web ACL's with
only locally defined keys.


### Linking a WebKey to a WebID

This goes beyond authentication, but can be integrated very simply in a
number of ways. 

The simplest way is for the WebID document to be the same
as the KeyId document. For example Alice's `WebID` and `keyID` documents
could be `https://alice.databox.me/profile/card` and return
a representation with the following triples:


```turtle
<#me> a foaf:Person;
    foaf:name "Alice";
    cert:key <#key1> .

<#key1> a cert:RSAPublicKey;
        cert:modulus "00cb25da76..."^^xsd:hexBinary;
        cert:exponent 65537 .
```                                         

Todo: how does one deal with `keyId` documents that are different
from `WebID` documents?
