# hypothesis

In considering the [prior art](prior-art.md), we notice that while the semantic web is focussed on data representation (and hence portability), it is agnostic, to a degree, to data _distribution_. It is based in the concepts of the Web, as a medium for _public_ data. Personal data stores are, of course, focussed on personal (and _private_) data, and make a strong statement about data distribution: it is via stores, which are (at least implicitly, and in all implementations) server-based. Local-first generalises the same concept of personal storage explicitly to clients; but in doing so, accepts the additional complexity of keeping multiple clients in-sync with each other, especially when collaborating (even privately). The ability to overcome this complexity is (proposed to be) dealt with using CRDT data structures.

Can we combine the advantages of all three approaches? In doing so, we need to reconcile these two apparent conflicts: between servers and clients, and between semantic web data representations and CRDTs.

We believe that neither of these are conflicts at all.

- Having solved client-to-client data distribution, client-to-server becomes a special case, in which one storage location happens to be always-on, and may have access to greater compute and storage capacity. This unifies personal data storage and local-first.
- Semantic web's data representations are not incompatible with CRDTs – we know this because we have been researching this with the technology **m-ld**, based on an RDF CRDT.

Therefore, our hypothesis in this project is that web app data securely stored in **replicated semantic graphs** can both exhibit the desirable properties of [Sustainable Apps](../sustainability.md) and meet today's and tomorrow's feature expectations without the high costs and limitations of today's distributed data architectures.

---

| ≪ prev: [prior art](prior-art.md) | next: [project approach](approach.md) ≫ |
|-----------------------------------|-----------------------------------------|