# Text API

> NB1: This specification is a generalisation of the [Explicit Updates syntax [1]](https://github.com/m-ld/m-ld-spec/wiki/Explicit-Updates) being developed for [Text CRDT Support [2]](https://github.com/m-ld/m-ld-spec/issues/35) in **m-ld**. That work is also part of the Sustainable Web Apps project.

> NB2: For the purposes of this specification, we would like to be agnostic to data serialisation; but in the interests of making our analysis concrete, we'll put examples in JSON-LD (or, when considering updates, [**json-rql** [3]](https://json-rql.org/)). The examples should be readily generalisable to other syntaxes.

> NB3: This page is currently organised as an analysis narrative. Feel free to cut to the [conclusion](#conclusion-splice). We'll probably move the narrative elsewhere once the specification is more elaborated. 

## updating text

In our commitment to using [replicated semantic (RDF) graphs](hypothesis.md) as a core data representation, we may naturally represent text using the [built-in string type [4]](https://www.w3.org/TR/rdf11-concepts/#xsd-datatypes):

```json
{
  "@id": "fred",
  "story": "Flintstones, meat the Flintstones"
}
```

Appending more text using **json-rql** (a dialect of [SPARQL [5]](https://www.w3.org/TR/2013/REC-sparql11-query-20130321)) requires to delete the previous value from the graph:

```json
{
  "@delete": { "@id": "fred", "story": "Flintstones, meat the Flintstones" },
  "@insert": {
    "@id": "fred",
    "story": "Flintstones, meat the Flintstones,\nThey're the modern stone-age family"
  }
}
```

We can remove the need to specify the old value by using a variable and the concatenation operator (this also translates into SPARQL, although with a little more complexity):

```json
{
  "@delete": { "@id": "fred", "story": "?story" },
  "@insert": {
    "@id": "fred",
    "story": { "@value": "?story", "@concat": "\nThey're the modern stone-age family" }
  }
}
```

Besides not repeating ourselves, matching the old value also expresses the semantic that we just want to append. If someone else has made a change in the meantime, we'll append to their value.

In order to maximise the smoothness of this common operation, we can further simplify the above by introducing a new top-level keyword that is an intuitive shorthand for 'change what was there before':

```json
{
  "@update": {
    "@id": "fred",
    "story": { "@concat": "\nThey're the modern stone-age family" }
  }
}
```

So long as the participants in this collaboration take strict turns, appending strings in this way is straightforward. However, it rapidly breaks down in cases of concurrent editing.

Let's say that someone corrects the typographic error in the initial story _at the same time_ as we append the second line. The result is this:

```json
{
  "@id": "fred",
  "story": [
    "Flintstones, meet the Flintstones",
    "Flintstones, meat the Flintstones,\nThey're the modern stone-age family"
  ]
}
```

What's happened? Both transactions matched the previous value, deleted it, and inserted a new value. But those new values are different. Because the transactions happened at the same time, the engine does not know which is correct, and presents a _Set_ of options (noting that in JSON-LD, a plain JSON array represents a Set unless otherwise specified).

In some cases the app might be coded to deal with this, for example by presenting the users with a drop-down box to select which conflicting value they want to keep. However, for a whole _document_ (such as in a word processor), that's clearly not the best user experience.

Much has been written about how to deal with such conflicts in such a way that both users' intentions are generally preserved (**and** everyone ends up with the same text), keeping the document coherent. One way is to use a text [Conflict-free Replicated Data Type (CRDT) [6]](https://crdt.tech). In this analysis, we'll allow that to be abstract. Our question here is: what should the application API be when such a CRDT is in use for our `story` property?

For our compositional artiste appending new lines to the story, the `@update` syntax with the `@concat` operator meets our needs. We didn't go into the syntax that our spelling wizard used, although its effect was also a delete-insert pair. One key difference is that the spelling correction was to a character in the middle of the text.

This can be achieved using [built-in string functions [5]](https://www.w3.org/TR/2013/REC-sparql11-query-20130321/#func-strings). To target a specific character requires the reconstruction of the string by concatenation:

```json
{
  "@delete": { "@id": "fred", "story": "?oldStory" },
  "@insert": { "@id": "fred", "story": "?newStory" },
  "@where": {
    "@bind": { "?newStory": { "@concat": [
      { "@substr": ["?oldStory", 1, 15] },
      "e",
      { "@substr": ["?oldStory", 17] }
    ] } }
  }
}
```

One of the reasons that this is awkward is that SPARQL (on which **json-rql** is based) is primarily designed as a declarative language – it should be agnostic to the underlying store's implementation. But using string functions it provides, the above becomes very procedure oriented. It's hard to imagine how an engine could reasonably reinterpret the intention of the update to be anything other than a concatenation of substrings – which is definitely not the intention we want to communicate to the CRDT, so that it can be preserved across concurrent changes, i.e. a single-character replacement.

For inspiration, let's look at the APIs that other text-handling systems use.

### Y.Text

The text datatype of the CRDT library [Yjs [7]](https://docs.yjs.dev/api/shared-types/y.text) provides individual `delete` and `insert` methods:

> `ytext.delete(index: number, length: number)`<br>
  Delete length characters starting from index.

> `ytext.insert(index: number, content: string[, format: Object<string,any>])`<br>
  Insert content at a specified index.

Both a delete and an insert is needed to correct the spelling. The two calls can be made atomic using a transaction on the parent document (a Y.Doc).

### codemirror

The editor library **codemirror** allows text state to be modified using a [`ChangeSpec` [8]](https://codemirror.net/docs/ref/#state.ChangeSpec):

> `type ChangeSpec = {from: number, to?: number, insert?: string | Text} |
ChangeSet |
readonly ChangeSpec[]`<br>
> This type is used as argument to EditorState.changes and in the changes field of transaction specs to succinctly describe document changes. It may either be a plain object describing a change (a deletion, insertion, or replacement, depending on which fields are present), a change set, or an array of change specs.

### diff-match-patch

The Diff Match and Patch libraries from Google describe text patches as _tuples_, which can be generated with a [diffing algorithm [9]](https://github.com/google/diff-match-patch/wiki/API#diff_maintext1-text2--diffs) or hand-crafted, and applied to text:

> ... describe the transformation of text1 into text2. Each difference is an array (JavaScript, Lua) or tuple (Python) or Diff object (C++, C#, Objective C, Java). The first element specifies if it is an insertion (1), a deletion (-1) or an equality (0). The second element specifies the affected text.<br>
> `diff_main("Good dog", "Bad dog") → [(-1, "Goo"), (1, "Ba"), (0, "d dog")]`
 
Note that this library is oriented to matching text in documents that may have changed in the meantime; hence the inclusion of both unchanged and deleted text in the patch to allow fuzzy matching. Other libraries have adapted the tuple format to allow replacement of these with integer character counts, for cases where the document is in a known state.

### m-ld lists

**m-ld** already supports convergent sequences by [supporting the JSON-LD `@list` keyword [10]](https://spec.m-ld.org/#lists). This is not intended for text; but it's possible to model text as simply a sequence of single-character strings, like this (truncated for space):

```json
{
  "@id": "fred",
  "story": { "@list": ["F", "l", "i", "n", "t", "s", "t", "o", "n", "e", "s"] }
}
```

Having done this, our single-character replacement uses a delete-insert syntax (taking care to target the story list with the `@where` clause):

```json
{
  "@delete": { "@id": "?story", "@list": { "15": "a" } },
  "@insert": { "@id": "?story", "@list": { "15": "e" } },
  "@where": { "@id": "fred", "story": "?story" }
}
```

### conclusion: `@splice`

Common features across our considered prior art (and many other examples) are
- target characters in the text by index
- delete sequences of characters in batch, using an integer length
- insert new sequences in batch, using a string

We also need to maintain **json-rql**'s declarative nature, while ensuring that the user's intention is precisely captured. We propose a `@splice` keyword, very similar to [Javascript's Array method [11]](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/splice):

```json
{
  "@update": {
    "@id": "fred",
    "story": { "@splice": [15, 1, "e"] }
  }
}
```

The parameters of `@splice` are typed as `[index: number, deleteCount: number, insertContent: string]`.

This has the advantage that many editor commands (typing a character, pasting text, deleting a selection) can all be captured in a single operation.

It's also possible to perform multiple operations in a single update:

```json
{
  "@update": {
    "@id": "fred",
    "story": [
      { "@splice": [15, 1, "e"] },
      { "@concat": "\nThey're the modern stone-age family" }
    ]
  }
}
```

## notifying updates

If a CRDT is in-use for the property, transacted updates at the local replica are transformed into CRDT operations for broadcast to other replicas; at the destination, they must be applied to the local CRDT state. How is the application notified of the update?

A normal **m-ld** API update notification is very similar to an update transaction, having `@delete` and `@insert` components indicating [property changes to Subjects [12]](https://spec.m-ld.org/#events). All updates, even splices, could be notified as such – deletion of the old text and insertion of the new. In this case there is no possibility of concurrency, because that is already resolved by the local engine, and all updates notified to the application are serial.

However, it's still awkward and inefficient to notify deletion and insertion of the _entire_ text. This might work for an HTML `textarea`, whose API only has set/get of the whole `value`; for a larger document in a dedicated text component (like [codemirror](#codemirror)), this could lead to unacceptable performance. In both cases, it's likely to destroy the local user's caret position, selection, and scroll position.

Instead, we choose to notify the update using the same `@update ... @splice` syntax as for the transaction. The expectation is that the local app is maintaining the text string in a local text-handling component (probably a UI), and can translate the splices to the local component API. If that's not suitable, it's always possible to recover the full, current text string from the replica using a query.

One complication in update notifications is that the CRDT operation may resolve to multiple splices in different locations, for example if the local app concurrently changed the text so that the splice, as mapped to the local text, is no longer contiguous. This scenario uses the multiple-operation variant above.

## declaring text algorithms

In alignment with the principle of decentralised extensibility, it should be possible to declare which collaborative text algorithm is in-use at a particular location in the graph. Ideally this should allow for various ways to target the location:

1. Any value for a property (predicate), e.g. `"story"`
2. A literal value with a specific datatype, e.g. [rdf:text [13]](https://www.w3.org/2007/OWL/draft/ED-rdf-text-20090420/)
3. A value for a property, where the Subject has a specific `@type`

The latter option is complicated by the fact that during update processing, the local engine may not have knowledge of the `@type` of every Subject being updated; and further, the type might even be changing concurrently.

Initially, therefore, we will allow for options 1 & 2 in our implementation. Declaring the CRDT algorithm will follow the normal pattern for [**m-ld** Extensions [14]](https://js.m-ld.org/#extensions), by writing the declaration to the graph, e.g.

```json
{
  "@context": {
    "mld": "http://m-ld.org/#",
    "js": "http://js.m-ld.org/#",
    "sh": "http://www.w3.org/ns/shacl#"
  },
  "@id": "http://m-ld.org/extensions",
  "@list": [
    {
      "@type": "js:CommonJSExport",
      "js:require": "@m-ld/m-ld/ext/tseq",
      "js:class": "TSeqExtension",
      "sh:path": { "@vocab": "story" }
    }
  ]
}
```

- In this example, `TSeqExtension` is a Javascript class applying a Text CRDT (under development).
- We have co-opted the SHACL `path` predicate to declare the affected property. 🚧 This approach may change.
- As usual, this declaration will be simplified with a utility method.

## bibliography
<br>[1] https://github.com/m-ld/m-ld-spec/wiki/Explicit-Updates
<br>[2] https://github.com/m-ld/m-ld-spec/issues/35
<br>[3] https://json-rql.org/
<br>[4] https://www.w3.org/TR/rdf11-concepts/#xsd-datatypes
<br>[5] https://www.w3.org/TR/2013/REC-sparql11-query-20130321
<br>[6] https://crdt.tech
<br>[7] https://docs.yjs.dev/api/shared-types/y.text
<br>[8] https://codemirror.net/docs/ref/#state.ChangeSpec
<br>[9] https://github.com/google/diff-match-patch/wiki/API#diff_maintext1-text2--diffs
<br>[10] https://spec.m-ld.org/#lists
<br>[11] https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/splice
<br>[12] https://spec.m-ld.org/#events
<br>[13] https://www.w3.org/2007/OWL/draft/ED-rdf-text-20090420/
<br>[14] https://js.m-ld.org/#extensions

---

| ≪ prev: [query API](xql.md) |
|-----------------------------|