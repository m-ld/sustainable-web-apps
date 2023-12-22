# Using an Ordered List

Trying out the Todo list app, you might have noticed that the items don't necessarily get added to the end of the list (they seem to end up in lexical order). This is because the underlying data model is a graph, which is an unordered Set of graph edges (the **m-ld** engine has an underlying lexical order, which should not be relied on). We have not yet told our app in which order to present the Todos.

We could add an index property to each Todo object, and always give new Todos an index which is the prior length of the list. Bearing in mind that other participants might be adding their own Todos to the same list, it's possible that items in the list could sometimes end up with the same index. No problem, we could code around this quite easily. But notice that we're now busy designing a collaborative data structure.

As you might expect, [collaborative ordered lists](https://edge.js.m-ld.org/interfaces/list.html) are provided as a data structure in **m-ld**. Let's use one.

First, we need to create a higher-level entity which represents the list itself. Change these lines:

```js
    model.write({
      title: newInput.value,
    }).catch(console.error);
```

To this:

```js
    model.write({
      '@id': 'todos',
      '@list': { 0: { title: newInput.value } }
    }).catch(console.error);
```

You might find it a little odd that we're writing the list object every time we add a new Todo. The important thing to realise is that every write to **m-ld** is always a _patch_: we're inserting new data, not overwriting old data. Because the graph as a Set, as we've already mentioned, adding an object identified as `'todos'` does nothing if it already exists.

The behaviour of the `'@list'` property is a bit more involved. A List can have the same item in it multiple times. What we're actually saying in this patch is that the new input value should be at position zero in the list – and anything else in the list should remain (even if it's the same item again), just shuffled one index down.

We're not done yet though, because in our query we're still reading the Todo items individually, not reading the ordered list. Instead of describing every Todo in the graph, we want to describe the list itself. However this presents a small problem: a Describe query presents _shallow_ objects only. We'll get the Todo identities as References, but not their titles. Instead, we want to ask **m-ld** for the Todo list with its _deep_ items. To do this, we use a [Construct query](https://edge.js.m-ld.org/interfaces/construct.html). Change these lines:

```js
const roq = watchQuery(model, state => state.read({
  '@describe': '?id', '@where': { '@id': '?id', title: '?' }
}));
```

To this:

```js
const roq = watchQuery(model, state => state.read({
  '@construct': {
    '@id': 'todos',
    '@list': { '?': { '@id': '?id', title: '?' }}
  }
}));
```

If you compare this syntax with the `write` above, you'll see that we're using a variable (`'?'`) in the list index position; so this pattern matches anything with a title that appears in the `todos` list at any index position.

One final change: our query now returns a single object, the list (although it's still wrapped in an array). We need to change the parameters to our subscribe handler, from this:

```js
roq.subscribe(todos => {
  //...
});
```

To this:

```js
roq.subscribe(([{ '@list': todos } = { '@list': [] }]) => {
  //...
});
```

This destructuring pulls out the `'@list'` property of the one returned object, with a suitable default of an empty array in case the `todos` list itself does not exist yet.

<a href="https://flems.io/#0=N4IgZglgNgpgziAXAbVAOwIYFsZJAOgAsAXLKEAGhAGMB7NYmBvAHgEIARAeQGEAVAJoAFAKIACEmQB8AHTQtJUMVAxoA5gF4ZIJttloZxBTAwATfYcMscxDGOqEMAJzgxiWkAFdiYALQAObTEAegtiKxs7TBwPADcIGAB3AAdaJ3CQe3pGBg9EiFNiQg1TGHjqGF98wsIKMQg0CGIIDChfOGpWmA0ARiDQuUsjZuJYKT4YLGSVRjFAIgIxPlpTWgBZADUeFmCRscGjYMITc32WACNlgE8wq1dqZvp7FTg4D2Jl2gxk5KCCjy+fiAbuEjEczDAnE8MC8PGDSk49PshlZCD0pCw7H9tIRaDg9O8VnBthh0Yc0UiQVYGslvBTkcRphgKjioPCPAB1RzEMRoGAwUxwMTvMRnGBiFa8gD82jplNs3loYFo1E8cFlQ2oz1e2l5iV8BNoMoMcqGWJAuqN9KGpggcAwZ1gpnVxAGxqG2zhEP0Yh9vpYdweaChMO0WAwDV+pg8YYjQLkvoTYhYniUmuh2pABt8UFtGXqUe0ObgGW9iYTLEYUxmYrNWcrjMYiKDZcTLBz+beHybLZbLBtsWDGfiSW7Pd7KlFSjNE5gUD02xnUFLY79wX7y7HC4gG7L23r1Z3q5Th+2AYg9A3p5g93PaDC2wupmucg9pCXckoIFcsBv9AQiBAABmAAGRBgJAABfCh0GwXAAPwAArBAqDoBgmGIPAICmNJuWATV6BgOpPE8AoILEMAnFxMQAHISAZOBEGCYJ+TUGBELgfAsGzUx8DSNRmIADxdBpSgEzikOogBuOQsNSdIxGAVZJjSS4ABkylnMiKKo2jiHoxjmNMVj2M47jeKcfiYCE4IcCwFTsw0qBxLgKSZOw+TgAASVoAAlZTGDgLTKKwGi6OSBimJYtikNM1lzMs6y4GVABrNx8HPZzXMadzcMSDBiAcABFTwIUuIKdLCiLDOMmKuLivjBJdJwxKwCTpIMNBUOLcVcXDIMNB5JIxAAVR81SAGUTCcBwhGcbA4AAChWFUcAYfAoGVfLb3wVxnAcABKfBWOIBbqJWGM0Go-b2q67lUMgNQxAG4B4xogABApqMQMRiIKBb9ooV7qLe86+q+nqLrEAAfKGxAAAwAEmAX7TH+iD8BpB0IGoI7Elini+LhwHm1Y3k4Ftb62FBhpiZ9c9vuRpwIG+uHKoMtQ8bqgmLLhiC5Agm6-25OzSiUAaMDypongIhbdTEJS7KcNTHLqby-LsgK6nuiA1GuuQutoWB1toNQFrh1ZllnewnBMRhTDEfKxCR7W1GQYHqcugBdCC4b1jrbsGxJPLQGluQG5bPFW4gjrcERYCjgAhS5PNR6jdSu9rdWD0P8BtO0HX5J7yNaVxM6SbPvHwMxTBEWJ0NU3MmAhU7UsuTxkmouoFpgfanqkBTXogMAxG7-BW6ejQBuokQ0KcaixAAMgXwOK+j2JWhK9amDUIpe5e5sfSYpNq6FI5Fg+DcRdnfBEiZxgFv3stgc+77qINFzacTYGi2IcHgDEUCCkhRNFgN9LOIdK7rygCVMQZE+YH1gYdToBVCALQNkbCElEnB+wTEfPgRwgyaimqfGsEDuQXAEuRNIJDBpCXqJWV6PpwE5ygTAqeWUfTwIgn7fWQsxCUQAI5Fzyig4qpUFpXygHUYs+UxQaH7jIxg+AbZmAfkDN6t0nCeHuH-Rh70X40Xfp3PR39cx-xopKcxz9TDg2opKT6dRdgwFfpY2B8DOH832n7AOP8i4RyjjHYgcdJjoSTinU6P8M68LQN1fcsi-HKkjuhQJwTE7J1TnWSYDYYBRLQIInangzgdCZqKBaC1UDvUid9d+sCi7-1McWcGyBPawM9r3eRA9mw+NzJ5SsgoBrv04l8BaBo+6dITAHCiGA1BRyLnEpRqEcjR3wryAAcpbEZWie7tQTFMmZyTjqpNCekiJ9pZxXXwIwISPBsjoTmR8S5ICYA7N9DbYgngnBBj2QEw58djnhLfh8XJnCcHKFzMomAjIKg8EINAUwNs0AjKydWOo+A0U-16ZMOAftuHtTkP4g5sc-kMDCanHEOALnV1rvXRuvInCnU1NjZKncHZwEuJ1Ee7T+6PyyDE7kuoOC9QaMIkStA8bJGCskE6JjBWQ2iGKNBjh1BimFHAfIKCIZ9X2iyl2bsQZCq9qCoeI8BUGsXsvU1kM2CT15Q9PVHtqJtPGb6fIaAVh4w2sg7a6YdaIrhpKD2GgkaWr6j7UFXC-afm-NeQM-4gL+DApBT2VAcxoGSv+VA5pYKYTdVZfA1AXifg+eQACbMmJOAlkdJohBCmqghIs5JdAsA2W4i21k+oPhYFiNQXw69GhQBUEhYINs7J13aO8G2wQC1wGCCJPN07PzEEuMkOCX4l2wCTRBIAA" target="_blank"><img src="flems.svg" height="30"> <b>Todo list update handling, on Flems</b></a>