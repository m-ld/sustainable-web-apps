# Removing a Todo List Item

Now that we have a model whose state is synchronised with the HTML UI, it's very straightforward to begin filling out the interactive features of our application. First, we'll add a button to remove each Todo item.

Add the following HTML under the <code class="hljs-name">label</code> having `id="label"` within the `todo-template`:

```html
<button class="destroy" id="destroy"></button>
```

When we create a Todo item, we wire up a listener for the button click in the new fragment created from the template:

```js
    fragment.getElementById('destroy').addEventListener('click', () => {
      model.write({
        '@delete': {
          '@id': 'todos',
          '@list': { '?': { '@id': todo['@id'], '?': '?' } }
        }
      }).catch(console.error);
    });
```

The syntax of the `write` might seem a little complex. The key thing to realise is that the value under the `'@delete'` key is a _pattern_: we wish to delete _matching_ graph content. What will match?

- `{ '@id': todo['@id'], '?': '?' }` will match all properties of the identified todo item (the variable in the property position means "any property", and in the value position means "any value").
- `{ '@id': 'todos', '@list': { '?': {/*todo*/} } }` will match any index position in the `todos` list with a todo matching the above (the variable in the index position means "any index").

Therefore, this write can be interpreted as "delete the todo with this identity, and remove its reference from the todos list".

There's nothing further to do: the deletion will appear as a new state without the Todo in our Reactive Observable Query, and this will be synchronised with the UI.

<a href="https://flems.io/#0=N4IgZglgNgpgziAXAbVAOwIYFsZJAOgAsAXLKEAGhAGMB7NYmBvAHgEIARAeQGEAVAJoAFAKIACEmQB8AHTQtJUMVAxoA5gF4ZIJttloZxBTAwATfYcMscxDGOqEMAJzgxiWkAFdiYALQAObTEAegtiKxs7TBwPADcIGAB3AAdaJ3CQe3pGBg9EiFNiQg1TGHjqGF98wsIKMQg0CGIIDChfOGpWmA0ARiDQuUsjZuJYKT4YLGSVRjFAIgIxPlpTWgBZADUeFmCRscGjYMITc32WACNlgE8wq1dqZvp7FTg4D2Jl2gxk5KCCjy+fiAbuEjEczDAnE8MC8PGDSk49PshlZCD0pCw7H9tIRaDg9O8VnBthh0Yc0UiQVYGslvBTkcRphgKjioPCPAB1RzEMRoGAwUxwMTvMRnGBiFa8gD82jplNs3loYFo1E8cFlQ2oz1e2l5iV8BNoMoMcqGWJAuqN9KGpggcAwZ1gpnVxAGxqG2zhEP0Yh9vpYdweaChMO0WAwDV+pg8YYjQLkvoTYhYniUmuh2pABt8UFtGXqUe0ObgGW9iYTLEYUxmYrNWcrjMYiKDZcTLBz+beHybLZbLBtsWDGfiSW7Pd7KlFSjNE5gUD02xnUFLY9bZ287yDaZDIFKxactEukY8u+I+8PQO2a+IG+XK6TwX7t57C4gT9bO0mDZgb592xTb+2AMIHoW9AJge5gLQMJLyufQPVIJc5EoEBXFgCD6AQRAQAAJgAVkQAA2bCQAAXwodBsFwLD8AAKwQKg6AYJhiDwCApjSblgE1egYDqTxPAKEixDAfcsDEAByEgGTgRBgmCfk1BgWi4HwLBs1MfA0jUeSAA8XQaUodNUujxIAbjkNjUnSMRgFWSY0kuAAZMpZyEkTcQkqTkhkuSFKUujVPUzSnG0mA9OCHAsAc7MXKgYy4DMiz2Os4AAElaAAJXsxg4Dc0TPOvbzZPk0xFOUwLWWC0LwrgZUAGs3HwYD4sSxpks4xIMGIBwAEVPAhS48o8yTCp8kqyoCtTKq03SXScIysBM8yDDQRji3FXFwyDDQeSSMQAFUMscgBlEwnAcIRnGwOAAAoVhVHAGHwKBlS6yD8FcZwHAASnwRTiBu8SVhjNBxO+5a1u5RjIDUMQduAeMJIAAQKcTEDEfiChu76KER8SkeBra0Y2kGxAAHzJsQAAMABJgEx0xsZI-AaQdCBqD+xIKo0rSqdx5tFN5OBbXRthCYafmfWA9H6acCB0apryxrULmpp5kKqZIuQSIhjDuSi0olB2jBOqaJ4eJu3UxDsqKnCc2K6nSrKopyupoYgNRwbkNbaFgZ7aDUG6qdWZZZ3sJwTEYUwxC6sQ6fdtRkHx8XQYAXRIqmvZWyHdsSVK0BpbkdvuzxHuIP63BEWAy4AIUuVLGfE3UweW3V88L-AbTtB1+Th4TWlcVuknb7x8DMUwRFiZjHNzJgIUBhrLk8ZJxLqG6YG+uGpBsxGIDAMR1-wRe4Y0HbxJEJinHEsQADIb9zkfy9iVp+uepg1CKTeEebH05KTcehRHEWB8W8BtZz4ESHLRgN1v5lnxqjdG4kDQJUlomfGRZiDE2AGIAADDLIUTRYDozbgXUez8oD9TEEJLWP8qG-U6N1QgN0fZ+whPuJwWcEx-z4EcTcsBnCAJrKQ7kFwdLCTSII3ael6iVkRj6EhHdyGULPq1H0NCSJZ29nrMQ+4ACOfdOqML6gNG6YCoB1GLF1MUGht6WMYPgCOZgYF4yRpDJwnh7hYLkcjBBElkGr28eg3MWCJKShCfA0wxNxKSlRnUXYMBEFhKoTQtR2tvpZxzhgvuJcy4V2IFXSYzE64N0BhgluWi0DrXrNWbJypS7MTyQU2u9dG51k-NWcpaA9EfU8GcDoctRQ3RuqgZGZT0bIKoX3bBQTizE2QKnKhqdN42J3s2TJuZUqVkFDtZBqkvg3QNFvVZCYc4iQwGoMufdqlWPwIxHI5duK8gAHKhwOe4jey0ExnIuQ0-6TSiktNKfaWcYN8CMD0jwbIzErkfDBYQmAnzfTfNyX86uAKSlA3gKeA8oLx6T2nrPXkThAaanZnVVeB9lnb1gYmMxECoEwGcbQhMydZxuBgF45laCUaRMQf41BPYZmYPwdE8JPLiYGiTuK1OdRRWJOvtQ7xCYUmJg0bcrqDhmEYV9kpNhaROG+g0Yin0EdiCeCcEGZFvzK5ooYMU1pHxOlqINRghxMBGQVB4IQaApgI5oAOe0qxdR8AhowZsyYcAs5GsGGgHJ1r8m2uIPawGOIcC4tMBPKeDAZ7FjnsS8SpLqDkrqNCS4q1KVHJpTnXUHBNoNAMQZWgXNkiiWSADQJtbSbRDFMwxw6gxTCjgPkRhJMtrfQpQnKVKdxJLMRXvA+Na61BjvrnTtW0xBsFPlkNAMMp1LpnV-bx+RY1Nv9gw966YPb+qppKFOGg6aLpBhnA16is7IVQuBQMmEQAABYACciAcGkRlSAHMaA6qYVQOaSirFY1hVuS8ZC5ryBYSVsVJwJs-pNEIL01UEI7kNLoFgCK6kSOsn1B8LAsRqC+Gfo0KAKg6LBAjlFKe7R3gR2CNQF4wQDLwe4-RTMlxkhURQsQS4sBgMkSAA" target="_blank"><img src="flems.svg" height="30"> <b>Todo list item removal, on Flems</b></a>