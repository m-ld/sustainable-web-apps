# architecture

## Components

Based on our [hypothesis](hypothesis.md), this is how we see a sustainable web app being structured, in the abstract.

![architecture components](img/architecture.component.svg)

- The JavaScript code and other static assets of the Sustainable Web App (the _app_, in pink) are served from some origin via HTTP.
- The app establishes a working Dataset, comprised of _semantic web_ data (not being too specific yet as to serialisation).
- The Dataset is stored locally, and so is available without a network round-trip (unless it has to be fetched to get started!). It is _local-first_.
- If other replicas of the Dataset exist elsewhere, they are synchronised, when the network is available, via a Message Delivery service.
- If desired for data safety, the Dataset can also be replicated to a Backup service, having its own persistent storage.
- Such a Backup service may also expose the data in the Dataset as Linked Data via its own API. This makes the data de-referencable, via HTTP, for more loosely-coupled consumers.

### Collaborative Web Library

The Collaborative Web Library (the _lib_) is primarily responsible for ensuring that the Dataset is as up-to-date as physics allows, compared to any other replicas that may exist. It exposes the Dataset to the application via an [API, comprising Reactive Observable Queries and Data Writing](xql.md).

> In providing an easy-to-use and robust programming model, this API is the substrate on which Sustainable Web Apps might be built. It should ultimately find its place among the standards on which the Web is built. We may eventually recommend that it be supported natively by browsers.

### Services

While local-first in principle, Sustainable Web Applications can still benefit from secure durable storage of data on a server. They also rely on a message delivery mechanism, which in practice must be either a deployed service (like a message broker) or a cloud service. While these are cheap to set up, they may become expensive in the long run due to operational costs, provider markup and service over-specification.

Both Message Delivery and Secure Backup are therefore key components for a Sustainable Web App, and should preferably be treated as _attached resources_, as recommended in Twelve-Factor Apps [1]. Further, they should be _generic_, that is, not specific to the app. This will encourage an ecosystem of providers, who can be chosen on the basis of merit and value; and swapped at will (no lock-in).

For secure backup, this design aligns closely with the concept of [Personal Data Stores](prior-art.md#personal-data-stores) (PDS), but notice that here, the primary function of the service is to provide data safety. It may _also_ provide its own API to serve as a vector for de-referencing the Dataset, thus potentially serving as a PDS, or even contributing to the Linked (Open) Data Cloud [2].

> 🚧 More detail will appear here as we work towards the Gateway milestone, as discussed in the [project approach](approach.md).

## Security

The security of a sustainable web app built according to this architecture is subject to a number of vulnerabilities and candidate controls in-common with the underlying web components upon which this design is based. These are mostly well-described online, a good starting point being the OWASP Top Ten [3].

Here we will address new or refined considerations arising from the _combination_ of components.

### User Authentication and Access Control

The Collaborative Web Library necessarily has to make a connection to the Message Delivery service; as a provided service, it is likely to require credentials. But as a local-first application, these credentials cannot safely be deployed with the application to the browser, where they would be subject to attack. So, they have to be issued to the _user_, who provides them to the app, and thereby to the lib. Secure issuance of such credentials is generally a solved problem with technologies such as WebID [4] and OpenID Connect [5]. However, setting this up is onerous (personal experience!) and so, not well aligned with our principle of easy-of-use...

> 🚧 More detail will appear here as we work towards the Security Support milestone, as discussed in the [project approach](approach.md).

# bibliography
<br>[1] https://12factor.net/backing-services
<br>[2] https://lod-cloud.net/
<br>[3] https://owasp.org/www-project-top-ten/
<br>[4] https://www.w3.org/wiki/WebID
<br>[5] https://openid.net/