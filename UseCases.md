# Authentication Use Cases

The following are use cases for authentication. The spec that is created by the authentication panel should have a technique to satisfy both the acquisition and validation of an identity proof for each of the scenarios.

It is possible that we decide some of these use cases should not be possible, but we must have a difinitive reason why that is.

When `platform` is seen, it should be substituted for examples of the following user-focused platforms:
 - Web-Browser based Application
 - iOS App
 - Android App
 - Desktop App
 - IOT device

## Use Cases
 - A user using a `platform` wants to at least access her Pod.
 - A user using a `platform` wants to at least access her friend's Pod.
 - A user using a `platform` wants to access both her and her friend's Pod.
 - A user using a `platform` and a self-signed private key wants to be able to authenticate and access Pods using that key.
 - A user using a `platform` and a self-signed private key wants to be able to authenticate and access Pods without using DNS.
 - A user wants to authenticate after moving her identity to a different URI
 - A remote application that the user is not present to interact with wants to access any number of Pods with a user's identity.
 - A remote application that the user is not present to interact with wants to access any number of Pods with its own identity.
 - A user wants to go on a remote camping trip with her friends. She brings along a mobile Pod that stands up its own network. Her and her friends want to access that Pod using the Solid identities they usually use on a `platform`.
 - A pod owner wants to set access controls such that only users with certain verifyable credentials can access the Pod, but the Pod does not need to know the user's full identity, only the credentials that pertain to its control.
 - A user using `platform` wants to authenticate using her company's SAML (or ActiveDirectory) IDP.
 - A user has an identity that is owned by another person (ie Parents owning children's identity).
   - Parents able to restrict use of the identity for certain resources
 - A user has multiple identities (ie work and personal) and wants to swap between them on `platform`.
 - A user has multiple identities (ie work and personal) and wants to access a Pod with whichever identity works
 - A resource requires the consent of multiple identities to gain access (for example accessing information on a joint mortgage)
