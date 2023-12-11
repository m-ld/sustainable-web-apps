# Security

The security of a Sustainable Web App is subject to a number of vulnerabilities and candidate controls in-common with the underlying web components upon which the [architecture](#reference-architecture) is based. These are mostly well-described online, a good starting point being the OWASP Top Ten [[OWASP]].

Here we specify new or refined considerations arising from the _combination_ of components.

## Identity and Access Control

The Collaborative Web Library necessarily has to make connections to the Message Delivery and Secure Backup services; as provided services, these are likely to require credentials. But as a local-first application, these credentials cannot generally be safely deployed with the static assets of the application, which are public according to the general principles of the web. So, logically they have to be issued to the _user_, who provides them to the app, and thereby to the lib.

Secure issuance of user credentials is generally a solved problem with technologies such as WebID [[WEBID]], OpenID Connect [[OIDC]] and Public Key Infrastructure (PKI) [[PKI]]. The main questions are: which existing technologies are best aligned with sustainability, and how best to securely and easily integrate them into a sustainable web app, without unduly compromising its principles?

To break down the problem let's label some axes, against which we can evaluate the suitability of technologies.

- **identity**: how are users identified to the technical components, and how is such identity proven?
- **ownership**: who "owns" an information Graph? Typically, this will be its creator, but ownership may change later. Note that ownership might be practically realised via a specific role in **access** control, but not necessarily.
- **access**: how do we control who can read and write a Graph? How is access control **bound** to the Graph such that it's not possible to bypass?
- **signing**: how are user identities bound to data; including, if necessary, to the specific updates they make, to enable auditing of app usage? (Note that we do not intend necessarily to imply cryptographic "digital" signatures here.)
- **logging**: where are app logs stored, including signed updates if applicable, and to whom is access to logs provided, for problem diagnosis and for auditing?

The first thing to note about these axes is that conventional approaches tend to be logically and physically _centralised_: Identity Providers (IdPs); cloud services mediating ownership and access control over user data; certificate authorities; and cloud monitoring and auditing services. This is contrary to our preference for local-first components. However:
1. While these functions apply to app data, counter-intuitively they do not have to be as available as the data itself, or co-located with it. This is because _all_ apps have always necessarily moved the data to the user for manipulation – you can't see data on a server – and so, existing tools are already designed around this characteristic. For example, PKI certificates [[PKI]] and Json Web Tokens (JWTs) [[RFC7519]] express claims about a user's identity and rights that are valid for a period of time and can be checked without reference to the issuing service.
2. Many technologies now available make important strides in the direction of the relevant sustainable principles: no lock-in, e.g. via the adoption of standards; and retention of control of personal data, e.g. via server-based personal data stores [[SOLID]].

For the purposes of this specification, then, we do not mandate any particular architectural choices for identity and access control, but instead allow apps to assess their choices using the axes above against the sustainability principles. Since not all technologies address all of our axes, there will frequently be a need to combine them. Combinations will entail additional complexities and trade-offs.

There follows an informal analysis of some example candidate technologies (a mix of standards and implementations), to illustrate how the axes can be used. Axes that are not addressed by the technology are omitted. The tag **≪partial≫** indicates that the axis is not addressed in full.

### Example: WebID [[WEBID]]

**Identity**

> The [WebID] Authentication Protocol authenticates a digital identity (a WebID) by allowing an Agent (e.g., a Web Browser) to prove possession of or access to a private key, whose corresponding public key is tightly bound to that WebID.

**Ownership ≪partial≫**

A WebID owns a user profile document. Ownership of arbitrary information Graphs can be canonically established by hosting of those Graphs on a website controlled by the WebID.

**Signing ≪partial≫**

A user with a WebID necessarily possesses a private key, with which it is technically possible to digitally sign arbitrary data. However, this is not a direct intention of the WebID specification.

### Example: OpenID Connect [[OIDC]]

**Identity**

> OpenID Connect 1.0 is a simple identity layer on top of the OAuth 2.0 [[RFC6749]] protocol. It enables Clients to verify the identity of the End-User based on the authentication performed by an Authorization Server, as well as to obtain basic profile information about the End-User in an interoperable and REST-like manner.

**Ownership ≪partial≫**

An End-User in OIDC owns a profile, which may be provided in part to the Client. 

**Access ≪partial≫**

OIDC is also based on, and so compatible with, OAuth:

> the OAuth 2.0 Authorization Framework [[RFC6749]] and OAuth 2.0 Bearer Token Usage [[RFC6750]] specifications provide a general framework for third-party applications to obtain and use limited access to HTTP resources.

### Example: Solid [[SOLID]]

> Solid is a specification that lets people store their data securely in decentralized data stores called Pods. Pods are like secure personal web servers for data. When data is stored in someone's Pod, they control which people and applications can access it.

**Identity ≪partial≫**

The Solid specification delegates identity management, allowing for either of the two above technologies.

**Ownership**

Solid establishes a strong ownership model over data by means of personal data _pods_, in which the data reside – at least, as the definitive source of truth. However, Solid pod servers are strictly limited to implementing the Solid specifications – by design, so that they are interchangeable. This means that they cannot be expected to participate in the additional protocol requirements of a local-first technology such as a CRDT.

However, data in Solid is inherently Linked Data, and in which every resource has a URL. This suggests an architecture in which access to the dataset is mediated via a server component, which is able to use the Solid Pod to record ownership of datasets ("deeds"). (Naturally, such a component can also "bank" quiescent data to the Pod when no-one is working on it. This has been proposed previously for **m-ld** domains.)

This approach has the drawback that the proposed app server component is not itself substitutable, and so manifests some lock-in. However, if this approach is found to be popular it could be standardised – as part of this initiative, or Solid itself.

**Access**

Continuing the above argument, Solid includes sophisticated access controls, which can be queried and enforced by the proposed mediator server component.

**Logging**

As a personal data store, a Solid Pod can be used for arbitrary data, including logs. The proposed mediator server component could be responsible for streaming the logs to Solid. Some previous work has been done on storing CRDT-like operations in a Solid Pod.

<div class="remove">

---

| ≪ prev: [architecture](architecture.md) | next: [query API](xql.md) ≫ |
|-----------------------------------------|-----------------------------|

</div>