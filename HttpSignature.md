# HTTP-Sig Authentication for SoLiD

[Signing HTTP Messages](https://tools.ietf.org/html/draft-ietf-httpbis-message-signatures-01)  (henceforth `HTTP-Sig`) is an IETF RFC Draft worked on by the HTTP WG, for signing and authenticating HTTP messages.
The work is based on [draft-cavage-http-signature-12](https://tools.ietf.org/html/draft-cavage-http-signatures-12), which evolved and gained adoption since 2013, being tested by a [large number of implementations](https://github.com/w3c-dvcg/http-signatures/issues/1), and this is set to grow by being taken up by the IETF.

HTTP Signature has the advantages of being very simple and being specified directly at the HTTP layer, bypassing the problem of client authentication at the TLS layer.
(Note: all communication here is assumed to run over TLS.) 

The protocol allows a client to authenticate by signing any of several HTTP headers with any one of its private keys.
In order for the server to verify this signature, it needs to know the matching public key.
This information must be transmitted by the client, in the form of an opaque string known as a `keyId` (see [§2.1.1 keyId](https://tools.ietf.org/html/draft-cavage-http-signatures-11#section-2.1.1)).
This string must enable the server to look up the key; how this look-up is done is not specified by the protocol.

This `Http-Sig` protocol extension allows the `keyId` to be interpreted as a URL.
The main use case is to allow the use of an `https` URL identifier ending with a fragment for the `keyId`, but it would also allow other URI schemes such as [DID](https://www.w3.org/TR/did-core/)s. 

By allowing URLs to be used in the `keyId` field,  we make it possible for the server to discover the key by fetching the `keyId Document`, whose URL is given by the [resolved](https://tools.ietf.org/html/rfc3986#section-5) `keyId` URL without the fragment identifier (see [§3 of RFC 3986: Uniform Resource Identifier (URI): Generic Syntax](https://tools.ietf.org/html/rfc3986?#section-3)).  

## Extending `HTTP-Sig` with URLs

Here we consider the minimum extension of `HTTP-Sig` with `keyId`s that are URLs.
We point to the advantages of this in terms of enabling client creation, editing and deleting of keys.
We then show how this ties into the larger Access Control Protocol used by Solid.

### The Sequence Diagram

The minimal extension to `Http-Sig` can be illustrated by the following Sequence Diagram:

```text
Client                          KeyID                            Resource
App                          Document                            Server
|                                |                                   |  
|-(1) request URL -------------------------------------------------->| 
|<==================(2) 40x + WWW-Auth Sig header + (Link) to ACL ===|
|                                |                                   |
| (choose key)                   |                                   |
|                                |                                   |
|-(3)- sign headers+keyId------------------------------------------->|
|                                |                       initial auth|
|                                |                       verification| 
|                                |                                   |
|                                |<-----------------(4) GET keyId----| 
|                                |-(5) return keyId doc------------->|
|                                                       - verify sig |
|                                                       - verify ACL |
|                                                                    |
|<---------------------------------------------(6) answer resource---|
```                                                                   

In (2) the Resource server responds to a request by sending a 401 or 402 response with the `WWW-Auth` Sig header.
It should also add a `Link:` relation to an Access-Control resource which describes which resources can have access. 
(Without such a Link the client would only be able to guess what key to send.)
With  [HTTP/2 server Push](https://tools.ietf.org/html/rfc7540#section-8.2), the server could immediately push the content of the linked-to Access Control document to the client, assuming reasonably that the client would have connected with the right key had it allready known the content. 
It may also be possible to send the ACL rules directly in the body (Todo: research) of the response.

The Access Control rule could be as simple as stating that the user needs to be authenticated with a key,  but any number of more complicated use cases are possible, as described in the [Use Cases and Requirements Document](https://solid.github.io/authorization-panel/wac-ucr/).

If the client can find a key that satisfies the Access Control Rules then it can use the private key to sign the headers in (3) as specified by [Signing HTTP Messages](https://w3c-ccg.github.io/did-method-key/) and pass a link to the key in the `keyId` field as a URL. 

The main protocol difference from `Http-Sig` is the potential request by the resource server for the `keyId document` in (4) to get that key. 
This may not necessarily lead to a new network connection being opened to the outside world in the following cases:
 * The `keyId` URL is local to the resource server,
 * The `keyId` URL is a [did:key](https://w3c-ccg.github.io/did-method-key/) URL, ie contains all the data of a public key,
 * The `keyId` URL refers to an external resource, but the resource server has a fresh cached copy of it. 
This can be the case of cached `https` URL documents but also for `did:...` documents stored on some form of blockchain which the server has access to offline.
 * The `keyId` is a relative URL on the client, which the server can GET using the P2P extension to HTTP (more on that below).

## Solid Use Case

In order to understand why we may want keys that are not local to the resource server we need to make a small disgression and explain the principal Solid use case.
We start by noticing that Solid (Social Linked Data) clients are modeled on web browsers. 
They are not tied to reading/writing data from one domain, but are able to start from any web server, and are able to follow links around the web. 
As a result, they cannot know in advance of arriving at a resource, what type of authentication will be needed there.
Furthermore they may be quite keen to create cross site identities and link them up, as that can allow decentralised conversations to happen and for people to build reputations across web sites.

We can illustrate this by the following diagram showing the topology of the data a solid client may need to read.
Starting from Tim Berners-Lee's [WebID](https://www.w3.org/2005/Incubator/webid/spec/identity/), a client may need to follow the links spanning web servers (represented as boxes).

![TimBLs foaf profile](https://raw.githubusercontent.com/wiki/banana-rdf/banana-rdf/img/WebID-foafKnows.jpg)

Starting from one resource, such as TimBL's WebID, a client should be able to follow links to other resources, some of which will be protected in various ways, requiring different forms of proof.

### The KeyId URL

In order for it to be clear that the `keyId` is to be interpreted as a URL, the `keyId` field MUST enclose the URL with `'<'` and `'>'` characters.
To take an example from [§A.3.2.1](https://tools.ietf.org/html/draft-ietf-httpbis-message-signatures-01#appendix-A.3.2) of `Http-Sig` this would allow the following use of relative URLs referring to a resource on the requested server

```HTTP
Signature-Input: sig1=(); keyId="</keys/test-key-a>"; created=1402170695
Signature: sig1=:cxieW5ZKV9R9A70+Ua1A/1FCvVayuE6Z77wDGNVFSiluSzR9TYFV
       vwUjeU6CTYUdbOByGMCee5q1eWWUOM8BIH04Si6VndEHjQVdHqshAtNJk2Quzs6WC
       2DkV0vysOhBSvFZuLZvtCmXRQfYGTGhZqGwq/AAmFbt5WNLQtDrEe0ErveEKBfaz+
       IJ35zhaj+dun71YZ82b/CRfO6fSSt8VXeJuvdqUuVPWqjgJD4n9mgZpZFGBaDdPiw
       pfbVZHzcHrumFJeFHWXH64a+c5GN+TWlP8NPg2zFdEc/joMymBiRelq236WGm5VvV
       9a22RW2/yLmaU/uwf9v40yGR/I1NRA==:
```

But it would also allow for absolute URLs referring to KeyId documents 
located elsewhere on the web, such as the requestor's [Freedom Box](https://freedombox.org):

```HTTP
Signature-Input: sig1=(); keyId="<https://alice.freedombox/keys/test-key-a>"; created=1402170695
Signature: sig1=:cxieW5ZKV9R9A70+Ua1A/1FCvVayuE6Z77wDGNVFSiluSz...==:
```

It could also allow key based did URLs as described in [issue 217 of the Solid Spec](https://github.com/solid/specification/issues/217#issuecomment-777509084).

The advantage of `https` URL in particular is that it allows the client to use HTTP Methods such as `POST` or `PUT` to create keys, as well as `PUT`, `PATCH` and `DELETE` to edit them, solving the problem of key revocation.

We also reserve the use of keyIds enclosed with `'>'` and `'<'` characters for possible extensions of HTTP such as the [Peer-to-Peer Extension to HTTP/2 draft](https://tools.ietf.org/html/draft-benfield-http2-p2p-02) as discussed on the [ietf-http-wg mailing list](https://lists.w3.org/Archives/Public/ietf-http-wg/2021JanMar/0049.html).
Here is an example of a header sent by a client to the server with such a URL:

```HTTP
Signature-Input: sig1=(); keyId=">/keys/test-key-a<"; created=1402170695
Signature: sig1=:cxieW5ZKV9R9A70+Ua1A/1FCvVayuE6Z77wDGNVFSiluSzR9TYFV
```

On receiving such a signed header, the server would know that it can request the key by making an HTTP `GET` request on the given relative URL on the client using the same connection but after switching client/server roles.
This would reduce to a minimum the reliance on the network.



### The KeyId Document

The `keyId` is a  URL that refers to a key.
An example key would be: 
  `https://bob.example/keys/2019-09-02#k1` 
The URL without the hash refers to the `keyId` document,
which can be dereferenced, so in the above case it would be
  `https://bob.example/keys/2019-09-02`

For the [Solid](https://solid-project.org/) use cases, the KeyId document would contain a description of the public key in an RDF format.
If we were to use [the cert ontology](https://www.w3.org/ns/auth/cert#) (as used by [WebID-TLS](https://dvcs.w3.org/hg/WebID/raw-file/tip/spec/tls-respec.html)) then this document would need to contain triples such as

```Turtle 
@prefix cert: <http://www.w3.org/ns/auth/cert#> .

<#k1>  a cert:RSAPublicKey;
     cert:modulus "00cb24ed85d64d794b..."^^xsd:hexBinary;
     cert:exponent 65537 .
```

(Other ontologies that could be used:
  * [The Security Vocabulary](https://web-payments.org/vocabs/security)
  * Any other?)
### The Access Control Rules

In order to understand a little bit better how the client can decide if it has the right key, we give a quick description of how Access Control Rules function.

The [Access Control Rules](https://solid.github.io/web-access-control-spec/) linked to by a resource, can specify an agent by describing their relation to a public key.

```Turtle           
@prefix  acl:  <http://www.w3.org/ns/auth/acl#>.
@prefix cert: <http://www.w3.org/ns/auth/cert#> .

<#authorization1>
    a             acl:Authorization;
    acl:accessTo  <https://alice.example/docs/shared-file1>;
    acl:mode      acl:Read,
                  acl:Write;
    acl:agent   [ cert:key <https://bob.example/keys/2019-09-02#k1> ],
                [ cert:key <https://candice.example/clefs/doc1#clef3> ]
```            

The keys can also be relative to the server of course (or `DID`s). 

Instead of listing agents individually in the acl they can also 
be listed as belonging to a group

```turtle
<#authorization1>
    a             acl:Authorization;
    acl:accessTo  <https://alice.example/docs/shared-file1>;
    acl:mode      acl:Read,
                  acl:Write, 
                  acl:Control;
    acl:agentGroup <https://alice.example.com/work-groups#Accounting> .   
```             

The agent Group document located at `https://alice.example.com/work-groups` can describe the users in various ways including by `keyId` as in this example 

```turtle
<#Accounting>
    a                vcard:Group;
    # Accounting group members:
    vcard:hasMember  [ cert:key <https://bob.example/keys/2019-09-02#k1> ],
                     [ cert:key <https://candice.example/clefs/doc1#clef3> ].
```

The Group resource can itself be access controlled to be only visible to members of the Group. 
Of course this requires adding members to the group to have as effect the sending of a notifications to the new member.

## Extending the Protocol with Credentials

We now look at how the protocol can be extended beyond possession 
of a key to prooving attributes based on a Credential.

### Protocol Extension for Credentials

A Credential is a document describing certain properties of an agent.
We can refer to such a document using a URL.

If the Access Control Rule linked to in (2) specifies that only agents that can prove a certain property can access the resource, and the agent has such credentials, it can choose the right credentials depending on the user's policies on privacy or security. 
If the policies do not give a clear answer, the user agent will need to ask the user -- at that time or later, but in any case before engaging in the request (3) - to confirm authenticated access.

Having selected a Credential, this can be passed in the response in (3) 
to the server by adding a `Credential:` header with as value a relative or absolute URL enclosed in `<` and `>` refering to that credential.
As before we reserve the option of enclosing a relative URL in `>` and `<` to refer to a client side resource if some form of P2P extension of
HTTP is available (see [Peer-to-Peer Extension to HTTP/2 draft](https://tools.ietf.org/html/draft-benfield-http2-p2p-02)). 

```text
Client                                Resource              KeyId           Age
App                                   Server                Doc             Credential
|                                        |                    |                |
|-(1) request URL ---------------------->|                    |                |
|<=======(2) 401 + WWW-Auth Sig header===|                    |                |
|                                        |                    |                |
| (select cert and key)                  |                    |                |
|                                        |                    |                |
|-(3) add Cred hdr+sign+keyId----------->|                    |                |
|                           initial auth |                    |                |
|                           verification |                    |                |
|                                        |                    |                |
|                                        |-(4) GET keyId----->|                |
|                                        |<-----(5) keyId doc-|                |
|                                        |                                     |
|                             verify sig |                                     |
|                                        |                                     |
|                                        |-(6) GET credential----------------->|
|                                        |<-----------------(7) credential doc-|
|                                        |
|                       WAC verification |
|                                        |
|<-------------------(8) send content----|
```

Steps (4) and (5), where the server retrives a (cached)copy of the key, are as before. 

Steps (6) can be run in parallel with (4) to fetch the Credential document.
This also can be cached. 
If the `Credential` header URL
* is enclosed in '<' and '>' then 
  * if it is relative it can be fetched on the same server
  * if is is an absolute URL it can be fetched by opening a new connection
* is enclosed in `>` and `<` and P2P HTTP extension is enabled, the server can request the credential from the client directly using the same connection opened in (3) exactly as above.

Note that if a client uses a P2P Connection to fetch a `>/cred1<` Credential and the client is authenticated with a `did:key:23dfs` then the server may be able to store that Credential at the `did:key:23dfs/cred1` URL in its cache. (todo: think it through)

### Linking a WebKey to a WebID

We start by illustrating this with a very simple example: that of authentication by [WebID](https://www.w3.org/2005/Incubator/webid/spec/identity/).
Here we consider a [WebID Document](https://www.w3.org/2005/Incubator/webid/spec/identity/#publishing-the-webid-profile-document) profile document to be a minimal credential - minimal in so far as it does not even need to be signed.
The signature comes from the TLS handshake required to fetch an `https` signed document placed at the location of the URL.

### WebID and KeyId documents are the same 

The simplest deployment is for the WebID document to be the same as the KeyId document. 
For example Alice's `WebID` and `keyID` documents
could be `https://alice.example/card` and return
a representation with the following triples:

```turtle
<#me> a foaf:Person;
    foaf:name "Alice";
    cert:key <#key1> .

<#key1> a cert:RSAPublicKey;
        cert:modulus "00cb25da76..."^^xsd:hexBinary;
        cert:exponent 65537 .
```                                         

By signing the HTTP header with the private key corresponding to the public key published at `<https://alice.example/card#key1>` the client proves that it is the referrent of `<https://alice.example/card#me>` according to the description of the WebID Profile Document.

This can be used for people or institutions that are happy to have public global identifiers to identify them.
One advantage is that the keyId document being the same as the WebID Profile document, the verification step requests (4) and (6) get collapsed into one request.
It also allows each individual user to maintain their profile and keys by hosting it on their server.
This allows friends to link to it, creating a [friend of a friend](http://www.foaf-project.org) decentralised social network.
A certain amount of anonymity can be regained by placing those servers behind Tor, using `.onion` URLs, and access controlling linked to documents that contain more personal information.

WebIDs allow servers to protect resources by listing WebIDs as shown in the [Groups of Agents](https://solid.github.io/web-access-control-spec/#describing-agents) description of the Web Access Control Spec.
Because the keys are controlled by the users, they can update them regularly, especially if they suspect a private key may have been compromised.

Authors of such ACLs can evaluate the trust they put in such a WebID by the position it has in their Web of Trust (i.e. which other people they trust link to it).

Access Control Lists can be extended by rules giving access to friends of a friend, extended family networks, ... (this is still being worked on) 

WebIDs are also useful for institutions wishing to be clearly identified when signing a [Verifiable Credential](https://www.w3.org/TR/vc-data-model/), such as a Birth Certificate or Drivers Licence Authority signing a claim, a University or School signing that a user has received a degree, ...

### WebID and KeyId documents are different

When WebID and KeyId documents are different this allows the key to be used without tying it to a WebID, and for that key to be used to sign other credentials.
It can also be useful in that the container where keys are placed can have less strict access control rules that the WebID profile, giving various software agents access to them.
In this case the WebID could link to the hash of the key, or some other proof that does not require it linking to the key.

### Credentials

Resource can describe in linked to `accessControl` document a class of agents specified by attribute.
For example ISO could publish a description at `https://iso.org/ont/People` describing the set of people over 21.

```Turtle
<#Over21> owl:equivalentClass [  a owl:Restriction;
      owl:onProperty :hasAge ;
      owl:someValuesFrom   
          [ rdf:type   rdfs:Datatype ;
            owl:onDatatype       xsd:integer ;
            owl:withRestrictions (  [ xsd:minExclusive     21 ]   [ xsd:maxInclusive    150 ] )
          ]
       ] . 
```

This would allow resources to be protected with a rule such as

```Turtle
<#adultPermission> 
          :accessToClass :AdultContent;
          :agentClass iso:Over21 ;
          :mode :Read .
```

A client after receiving the response (2) in the above sequence diagram can search for the relevant [Verifiable Credential](https://www.w3.org/TR/vc-data-model/) in its credential store (containing perhaps a Drivers Licence, Birth certificate or MI6 007 licence to kill), order these in a privacy lattice, and choose the one the more appropriate for the task at hand.
The URL for that Credential can then be sent in the header (3).
The verification process then needs to verify that the signature is correct, and that the credential identifies the user with the same key, and is signed by a legitimate Certificate Authority.
How to determine which Certificate Authority are legitimate for which claims, is outside the scope of this specification. 
This is known in the Self-Sovereign Identity space as a Governance Framework, and in the end will lead to a [Web of Nations](https://co-operating.systems/2020/06/01/WoN.pdf).
