# Using an Ordered List

Trying out the Todo list app, you might have noticed that the items don't necessarily get added to the end of the list (they seem to end up in lexical order); and also you can't add the same thing twice. This is because the underlying data model is a graph, which is an unordered Set of graph edges (the **m-ld** engine has an underlying lexical order, which should not be relied on). We have not yet told our app in which order to present the Todos.

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
      "@id": "todos",
      "@list": { 0: { title: newInput.value } }
    }).catch(console.error);
```

You might find it a little odd that we're writing the list object every time we add a new Todo. The important thing to realise is that every write to **m-ld** is always a _patch_: we're inserting new data, not overwriting old data. Because the graph as a Set, as we've already mentioned, adding an object identified as `"todos"` does nothing if it already exists.

The behaviour of the `"@list"` property is a bit more involved. A List can have the same item in it multiple times. What we're actually saying in this patch is that the new input value should be at position zero in the list – and anything else in the list should remain (even if it's the same item again), just shuffled one index down.

We're not done yet though, because in our query we're still reading the Todo items individually, not reading the ordered list. Instead of describing every Todo in the graph, we want to describe the list itself. However this presents a small problem: a Describe query presents _shallow_ objects only. We'll get the Todo identities as References, but not their titles. Instead, we want to ask **m-ld** for the Todo list with its _deep_ items. To do this, we use a [Construct query](https://edge.js.m-ld.org/interfaces/construct.html). Change these lines:

```js
const roq = watchQuery(model, state => state.read({
  "@describe": "?id", "@where": { "@id": "?id", title: "?" }
}));
```

To this:

```js
const roq = watchQuery(model, state => state.read({
  "@construct": {
    "@id": "todos",
    "@list": { "?": { "@id": "?id", title: "?" }}
  }
}));
```

One final change: our query now returns a single object, the list – although still wrapped in an array. We need to change the parameters to our subscribe handler, from this:

```js
roq.subscribe(todos => {
  //...
});
```

To this:

```js
roq.subscribe(([{ "@list": todos } = { "@list": [] }]) => {
  //...
});
```

This destructuring pulls out the `"@list"` property of the one returned object, with a suitable default of an empty array in case the `todos` list does not exist yet.

<a href="https://flems.io/#0=N4IgZglgNgpgziAXAbVAOwIYFsZJAOgAsAXLKEAGhAGMB7NYmBvAHgEIARAeQGEAVAJoAFAKIACEmQB8AHTQtJUMVAxoA5gF4ZIJttloZxBTAwATfYcMscxDGOqEMAJzgxiWkAFdiYALQAObTEAegtiKxs7TBwPADcIGAB3AAdaJ3CQe3pGBg9EiFNiQg1TGHjqGF98wsIKMQg0CGIIDChfOGpWmA0ARiDQuUsjZuJYKT4YLGSVRjFAIgIxPlpTWgBZADUeFmCRscGjYMITc32WACNlgE8wq1dqZvp7FTg4D2Jl2gxk5KCCjy+fiAbuEjEczDAnE8MC8PGDSk49PshlZCD0pCw7IQnDAwB58Hp3is4NsMOjDmikSCrA1kt5KcjiNMMBVCLQoPCPAB1RzEMRoGAwUxwMTvMRnGBiFb8gD82npVNs3loYFo1E8cHlQ2oz1e2n5iV8hNocoMCqGfz1SRNDKGpggcAwZ1gpk1xAGpqG2zhEP0Yj9-pYdweaChMO0WAwDV+pg8EajQLk-qTYhYniU2uhupARt8UHtGXqMe0ebgGV9yaTLEYUxmEot2Y+hsmTMYiJDFeTLDzhbeHzbHY7LDtsVDWfiVoT7YHFa7jpgSnrKnFUD02yX8-L04DwWHm+na4ge5nO2btaP27T5+2QYg9D315g91vaDC2wupmuci9pCg+koIFcWAn3oBBEBAHp-EQABmXwIMQHoACYQAAXwodBsFwMD8AAKwQKg6AYJhiDwCApjSXlgG1egYDqTxPAKZCxDAJxaCwMQAHISEZOBEGCYJBTUGAcLgfAsFzUx8DSNR+IADzdBpShk0TcPYgBuORSNSdIxGAVZJjSS4ABkynnRjmNYjiuOSHi+IEoTcNE8TJKcaSYDk4IcCwAzcxMqBlLgNSNLI7TgAASVoAAlfTGDgMyWLYzjiG43j+NMQThMc9lnNc9y4FVABrNx8FvfzAsaYKKMSDBiAcABFTwIUuOKLMS5LbLS+yRLErKpNkt0nCUrAVPUgw0AI0ssjQSA1DEDQdMTMRtAAAQKbREDEOiCgACgASgoBblpWOMDCQMQAAMABJgE20xduQ-BaSdCBqHwNREkyiSpLO-b20E-k4HtdbiCcBqfr9W91uupwIHWs6rJs4I3o+7KzuQuRkJG8beS80olDmjAqqaJ5qK2-UxD0rynCM3y6nCqKvJiuoCOmnbMZAtkhKgWg1C2s7VmWed7GxarBTEaqxCu5mIDUZB2KWo7IzQdiAF1kLO1nBjGkDeX1UK0FpXk5pWNUcAYV63BEWBTeIAAhS5Qtuy1Em0DW0F1-XvHwO0HSdUW5rAVpXBGuR3YN-AzFMERYiIwz8yYCEtu0QrLk8QE6i2mAdtmqR5vbCAwDEDP8GT2aNDm7QREIhFMgAMhrvkkj1sPYlaBr8FgdQiiz4AFr9HH53wRJocYLae6nf1ltW07tCNDVKF7ieQCWksMkhsQAAY192GB1tDz2W6gBqxEYtHx+Qnb8E6GrCC28aOfwCEWKcV2kz4xYjhDbUTEhIo6w93kLgySYmkEURwG5yXqNWBee9iD4APkfcuIBtAjT9Kfc+wctZoAmixAAjrNMQVVr71UaltfuUA6ilhFtnMQlDGD4GFrdMefplpYxBvcNauckyTxdNPBsRJtBg0XsvfMHDgCLRALKU6YjuEcO0NKKedRt7rTkUEZCp9UHox2q7OQWNlD5nwcbTw1tzbEEtpMIidsHaJxACvF2bMsG8mrC2CURtVRGKIiYsx1tLGOz4bQJsNYRZ2LkLg-AcBPBnA6NDcUW0tqoHEcI0sHDZ7H3wdIpetjTrIGVsfZWWcNA5yYZNCaK9QrVmFHNWeokvhbSNNQopfpdHMQwGoa2+CnG1kvtkDxVF+QADkBa1JBpnFB-pmmtI8YJUxVsLH218euFcIAL6MDkjwbpDB2kfHwNvUZfpsTEE8E4EM4zjFTK8bMqxM8+xLNGeghaK96EwCZBUHghBoCmGxGgWpp4RZ1HwP80p5TXboP-IBR8wZQIgCggAdkQOvXwMLEAAFYACcKFlZUDzGgfKoFUAgGiJhEACk3KXxeP+Q55AwLwxSk4Amr0miEAieqCEBEciwLoFgDy4kuXskNB8LAsRqC+Bbo0KAKhcLBGxF5aO7R3jYmCNQF4wRiVKUVXhbMlxkiEtLJcWA6LkJAA" target="_blank"><img src="flems.svg" height="30"> <b>Todo list update handling, on Flems</b></a>