# Pattern Definition

The Shared-Graph Model Pattern is an architectural pattern that extends Model-View-Controller [[MVC]], as follows.

- The Model is an RDF Graph [[RDF11-CONCEPTS]].
- The Model is _replicated_ to app instances sharing the same data.
- The Model replicas are [[[EVENTUALLY-CONSISTENT]]].
- The Model is _Reactive_ [[FRP]], such that both local and remote updates are expeditiously notified to the Controller & View.
- The Model should use RDF Vocabularies to declare its [=semantics=].
- The Model may be durably stored locally to the app.
- The Model may be durably stored in a backup location independently of the app.

Declared <dfn>semantics</dfn> include, but are not limited to:

- Data types, such as numbers, booleans, and strings
- Schema constraints, such as mandatory data and uniqueness
- Concurrency models, such as Conflict-free Replicated Data Types and locking expectations