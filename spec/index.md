# contents

1. [Introduction](index.md)
2. [Motivating Use-Cases](use-cases.md)
3. [Sustainability for Web Apps](sustainability.md)
4. Technical Design
   1. [Prior Art](design/prior-art.md)
   2. [Hypothesis](design/hypothesis.md)
   3. [Project Approach](design/approach.md)
   4. [Architecture](design/architecture.md)
   5. [API](design/xql.md)

# introduction

Goal: to encourage and support the development of Healthy Web Apps, defined by:

- No lock-in
- No attention theft
- Retention of control over personal data

So, why do people keep building apps which lock users in and keep control over
their users' personal data? We see two reasons.

First: there is an economic incentive to developing unhealthy apps. Data is
valuable, and companies who produce applications would prefer to have access to
as much of it as possible. Active users are valuable, as they produce valuable
data, and are also available to target with another revenue source, advertising.
Thus, the more an application developer can keep users and their data tied to a
platform, the more that application will succeed economically—that is, until and
unless the users demand an alternative. However, it's hard for users to make
much progress simply asking for change; the most effective way for users to make
their needs known is to actually leave the services they disagree with in favor
of those which treat them better, and to do that, there must be alternatives
available to move _to_.

Thus the second reason: there is a technological disincentive to developing
healthy apps. It's simply much easier to develop an application in which
everyone's data is centralized in a database under the application developer's
control. Exposing that data via some sort of API is a good deal of extra effort
that can be hard to justify. Interchanging data with other applications, to the
point of eliminating lock-in, is even harder to justify, as it both gives users
a route to leave the application _and_ is even harder to build than simply
exposing the data in some ad-hoc format.

We know there are people interested in developing healthy apps because they
currently do so, even in the face of technological difficulty. The entire Solid
project is evidence of this. We have a great deal of respect for Solid and its
approach, and draw inspiration from it. However, we believe it doesn't provide
enough to make a range of modern applications easy to build well: in particular,
apps with real-time multi-user collaboration.

We therefore propose a set of tools which make real-time multi-user
collaboration significantly easier to implement without lock-in or forcing users
to cede control of their personal data.

We will build these tools on top of m-ld's "domains", a CRDT for replicating
Linked Data graphs in realtime. By replicating the data locally among
collaborating participants, we preserve the users' ownership of their own data.
By using Linked Data as the basis for the data model, we ensure that application
data is readable at some level by any other Linked Data tool. For instance, the
data can easily be moved between a m-ld domain and a Solid pod if desired, where
it can be used by other applications. For hosted persistence, we offer the m-ld
Gateway, a server which (among other functions) operates as another participant
in the domain, and thus keeps its own copy of the data, analogous to an IPFS
pinning service. This Gateway service may be offered by the application
developer for convenience, but it may also be operated by end user themselves if
desired, preserving user control. Just as end users can bring their own pods to
a Solid app, they can also bring their own Gateways to these collaborative apps.

# background (how we got here)

| Doc                                                                                                                              | Published/Last Updated | Purpose                                                                       |
|----------------------------------------------------------------------------------------------------------------------------------|------------------------|-------------------------------------------------------------------------------|
| [Web Zero, with m-ld](https://github.com/m-ld/web-zero/wiki/Proposal)                                                            | 01-02-22               | Original Web Zero Proposal                                                    |
| [Questions](https://github.com/m-ld/web-zero/wiki/Questions)                                                                     | 30-03-22               | Feedback questions on proposal                                                |
| [Web Zero with RDFa “Spec”](https://github.com/m-ld/web-zero/blob/main/spec.md)                                                  | 20-02-23               | Emphasis on Linked Data rather than plain DOM                                 |
| [Web Zero Demo](https://github.com/m-ld/sustainable-web-apps/wiki/Web-Zero-Demo)                                                 | 21-02-23               | Web Zero 2023 concept (low m-ld specificity)                                  |
| [Web Zero (without DOM) (WIP).md](https://gist.github.com/Peeja/0212d1a299d78d92ddf3d5563e6bd1f9)                                | 08-03-23               | Justifying framework compatibility instead of DOM manipulation in the project |
| [Web Zero, with m-ld.delta](https://docs.google.com/document/u/1/d/1DrMr_PI6P81w1spiUDBTkOJGz0bPS2sZIXhCz51p7CE/edit)            | 08-05-23               | Justification change of direction, rolling up parts of 3-5                    |
| [Sustainable Web Apps, with m-ld.plan](https://docs.google.com/document/u/1/d/17YnR6f8Xp69E09sO3BShSP5H6PXx3sDGNTVxNyvl56s/edit) | 24-05-23               | Final plan                                                                    |
