# Reference Architecture

Expanding upon the pattern definition, this is how a web app using the Shared-Graph Model Pattern is structured, in the abstract.

![architecture components](design/img/architecture.component.svg "Architecture components")

- The JavaScript code and other static assets of the Web App (the _app_, in pink) are served from some origin via HTTP.
- The app establishes a working Graph Model, comprised of _semantic web_ data [[RDF-NOT]] (non-specific serialisation).
- The Graph is stored locally, and so is available without a network round-trip (unless it has to be fetched to get started!). It is local-first [[LOFI]].
- If other replicas of the Graph exist elsewhere, they are synchronised, when the network is available, via a Message Delivery service.
- If desired for data safety, the Graph can also be replicated to a Backup service, having its own persistent storage.
- Such a Backup service may also expose the data in the Graph as Linked Data [[LINKED-DATA]] via its own API. This makes the data de-referencable, via HTTP, for more loosely-coupled consumers.

## Shared-Graph Library

The Shared-Graph Library is primarily responsible for ensuring that the Graph is as up-to-date as physics allows, compared to any other replicas that may exist. It exposes the Graph to the application via a local API.

As part of this responsibility, the library will also enact the data constraints and concurrency models declared in the Graph. This means that the _available_ constraint vocabularies are likely to be library-specific. However, it is incumbent on the library provider to use standardised vocabularies where possible and publish any proprietary vocabularies, to facilitate data portability.

## Services

While local-first in principle, the Web App can still benefit from secure durable storage of data on a server. It also relies on a message delivery mechanism, which in practice must be either a deployed service (like a message broker) or a cloud service. While these are cheap to set up, they may become expensive in the long run due to operational costs, provider markup and service over-specification.

Both Message Delivery and Secure Backup are therefore key components for a Shared-Graph Model, and should preferably be treated as _attached resources_, as recommended in Twelve-Factor Apps [[12FA]]. Further, they should be _generic_, that is, not specific to the app. This will encourage an ecosystem of providers, who can be chosen on the basis of merit and value; and swapped at will (no lock-in).

For secure backup, this design aligns closely with the concept of Personal Data Stores (PDS) [[SOLID]], but notice that here, the primary function of the service is to provide data safety. It may _also_ provide its own API to serve as a vector for de-referencing the Graph, thus potentially serving as a PDS, or even contributing to the Linked (Open) Data Cloud [[LOD]].

<div class="remove">

---

| ≪ prev: [project approach](approach.md) | next: [security](security.md) ≫ |
|-----------------------------------------|---------------------------------|

</div>