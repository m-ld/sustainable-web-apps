# Introduction

> The World Wide Web, a "collaborative medium, a place where we can all meet and read and write" – Sir Tim Berners-Lee, 2005

This vision still lies at the heart of the Web. But web-based _apps_, particularly those offering user contributions and collaboration, have become associated with unwanted side effects like user lock-in, attention theft, and abdication of control over personal data.

The _Shared-Graph Model Pattern_ is a software pattern intended to expedite the development of healthier apps: those without dependencies on specific service providers, with user empowerment in terms of service and data portability, and with linking of data between apps – including apps developed against similar technologies sharing these principles.

Technically, it combines ideas from the semantic web [[RDF-NOT]] (machine-readable publishable interlinked data), personal data stores [[PDS]] (user control of user data) and local-first software [[LOFI]] (collaboration without obligatory third parties).

## Motivation

Why do so many existing apps lock-in their users and their attention, and keep control over their users' personal data? We see two reasons.

First: there is an economic incentive to developing apps with these properties. Data is valuable, and companies who produce applications would prefer to have access to as much of it as possible. Active users are valuable, as they produce valuable data, and are also available to target with another revenue source, advertising. Thus, the more an application developer can keep users and their data tied to a platform, the more that application will succeed economically—that is, until and unless the users demand an alternative. However, it's hard for users to make much progress simply asking for change; the most effective way for users to make their needs known is to actually leave the services they disagree with in favor of those which treat them better, and to do that, there must be alternatives available to move _to_.

Thus, the second reason: there is a technological incentive. It's simply much easier to develop an application in which everyone's data is centralized in a database under the application developer's control. Exposing that data via some sort of API is a good deal of extra effort that can be hard to justify. Interchanging data with other applications, to the point of eliminating lock-in, is even harder to justify, as it both gives users a route to leave the application _and_ is even harder to build than simply exposing the data in some ad-hoc format.

## Prior Art

We know there are people interested in developing healthier apps because they currently do so, even in the face of technological difficulty. The Shared-Graph Model Pattern is inspired by several existing projects providing visions or frameworks for such apps; and is compatible with them in principle or in implementation.

## Principle

The Shared-Graph Model Pattern is based on the idea that web app data securely stored in **replicated semantic graphs** can make it possible for app developers to meet today's and tomorrow's feature expectations without the high costs and limitations of today's distributed data architectures.

Using this pattern, the replicated (shared) semantic graph takes the role of the user interface Model [[MVC]], which is combined with app code implementing views and controllers, as applicable to the framework in use.
