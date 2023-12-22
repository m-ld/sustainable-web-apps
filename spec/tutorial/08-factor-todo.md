# Factoring the Todo Controller

As our application becomes more complex, we'll want to factor views and their controllers into discrete components. This will help engineers with their comprehension of the app structure, and provide subunits for testing. It may also help optimise the movement of data between the model and the view, particularly if parts of the model are not visible due to filters or paging.

While the boundaries between factorised components may be largely the same, the code structure will vary depending on the JavaScript UI framework or library you are using. Here, we'll make a simple factorisation using JavaScript classes, and illustrate how Reactive Observable Queries can be factored in parallel with the component structure.

You may have already felt that the subscription to the `roq` observable was getting unwieldy. Let's factor out the creation of a Todo list item into a class. The constructor will handle creating the HTML view:

```js
class Todo {
  static template = document.getElementById('todo-template');
    
  constructor({ '@id': id, title }) {
    const fragment = Todo.template.content.cloneNode(true);
    fragment.getElementById('label').textContent = title;
    fragment.getElementById('destroy').addEventListener('click', () => {
      model.write({
	  	'@delete': {
	      '@id': 'todos',
	  	  '@list': { '?': { '@id': id, '?': '?' } }
	  	}
	  }).catch(console.error);
    });
    this.root = fragment.getElementById('todo');
  }
}
```

The observable subscription can now be simplified to this:

```js
roq.subscribe(([{ '@list': todos } = { '@list': [] }]) => {
  list.replaceChildren(Todo.template, ...todos.map(todo => new Todo(todo).root));
});
```

This is an improvement. But the new Todo class still relies on its creator to pass in the Todo details to display. It would be better for it to encapsulate its own data dependencies.

We can achieve this with another Reactive Observable Query, inside the Todo, which loads its details (at the moment, just the title) from the graph, and keeps them up to date. We could use `watchQuery` again, with a Describe query that just loads the Todo with the given ID. However, there's a utility for this exact scenario, which is simpler, called `watchSubject`. As the name implies, this creates a reactive observable that just tracks the state of one identified Subject.

Remove the `title` parameter from the Todo constructor, and replace the line that sets the label text content with this:

```js
    const label = fragment.getElementById('label');
    this.subs = watchSubject(model, id).subscribe(todo => {
      label.textContent = todo.title;
    });
```

Now, we no longer need to load the Todo titles with a Construct query; we can switch to a simple Describe of the `todos` list. This will only contain [References](https://www.w3.org/TR/json-ld11/#dfn-node-reference) in the `@list` property, which is exactly what we want. And yes, now we can actually use `watchSubject` for the `todos` list as well! Replace the `roq` initialisation _and subscription_ with this:

```js
watchSubject(model, 'todos').subscribe(({ '@list': refs }) => {
  list.replaceChildren(Todo.template, ...refs.map(ref => new Todo(ref).root));
});
```

You'll find that now we actually have less code than before this step; and we've nicely encapsulated the Todo controller.

However, there's a subtle problem. By subscribing to each Todo subject, we've created a memory leak: these subscriptions will keep watching the model even after the relevant Todo has been removed from the view. You might have noticed that we captured the Subscription as `this.subs` following the call to `watchSubject`. This allows us to create a "destructor" for the Todo class, which calls `this.subs.unsubscribe()` (see the [RxJS Subscription documentation](https://rxjs.dev/api/index/class/Subscription)). Unfortunately, we then need to keep track of the Todo instances we've created. We'll leave this as an exercise for the reader; you can find a finished solution in the Flems link below.

<aside class="note">

When you use a JavaScript UI framework, it will typically provide a canonical way to have a destructor called for you.

</aside>

<a href="https://flems.io/#0=N4IgZglgNgpgziAXAbVAOwIYFsZJAOgAsAXLKEAGhAGMB7NYmBvAHgEIARAeQGEAVAJoAFAKIACEmQB8AHTQtJUMVAxoA5gF4ZIJttloZxBTAwATfYcMscxDGOqEMAJzgxiWkAFdiYALQAObTEAegtiKxs7TBwPADcIGAB3AAdaJ3CQe3pGBg9EiFNiQg1TGHjqGF98wsIKMQg0CGIIDChfOGpWmA0ARiDQuUsjZuJYKT4YLGSVRjFAIgIxPlpTWgBZADUeFmCRscGjYMITc32WACNlgE8wq1dqZvp7FTg4D2Jl2gxk5KCCjy+fiAbuEjEczDAnE8MC8PGDSk49PshlZCD0pCw7H9tIRaDg9O8VnBthh0Yc0UiQVYGslvBTkcRphgKjioPCPAB1RzEMRoGAwUxwMTvMRnGBiFa8gD82jplNs3loYFo1E8cFlQ2oz1e2l5iV8BNoMoMcqGWJAuqN9KGpggcAwZ1gpnVxAGxqG2zhEP0Yh9vpYdweaChMO0WAwDV+pg8YYjQLkvoTYhYniUmuh2pABt8UFtGXqUe0ObgGW9iYTLEYUxmYrNWcrjMYiKDZcTLBz+beHybLZbLBtsWDGfiSW7Pd7KlFSjNE5gUD02xnUFLY9bZ287yDaZDIFKxactEukY8u+I+8PQO2a+IG+XK6TwX7t57C4gT9bO0mDZgb592xTb+2AMIHoW9AJge5gLQMJLyufQPVIJc5EoEBXFgCD6AQRAQAAJgAVkQAA2bCQAAXwodBsFwLD8AAKwQKg6AYJhiDwCApjSblgE1egYDqTxPAKEixDAfcsDEAByEgGTgRBgmCfk1BgWi4HwLBs1MfA0jUeSAA8XQaUodNUujxIAbjkNjUnSMRgFWSY0kuAAZMpZyEkTcQkqTkhkuSFKUujVPUzSnG0mA9OCHAsAc7MXKgYy4DMiz2Os4AAElaAAJXsxg4Dc0TPOvbzZPk0xFOUwLWWC0LwrgZUAGs3HwYD4sSxpks4xIMGIBwAGVPDOGjwOIPKPMkwqfJKsqArUyqtN0l0nCMrATPMgw0EY4txVxcMgw0HkkjEABVDLHJ6kwnAcIRnGwOAAAoVhVHAGHwKBlS6yD8FcZwHAASnwRTiFu8SVhjNBxJ+1aNu5RjIDUMQ9uAeMJIAAQKcTEDEfiClun6KCR8TkZBnb0a20GxAAH3JsQAAMABJgCx0wcZI-AaQdCBqH+xIKo0rTqbx5tFN5OBbQxtgiYaAWfWAjGGacCAMepryJrUbmZt5kLqZIuQSMhjDuSi0olD2jBOqaJ4eNu3UxDsqKnCc2K6nSrKopyuoYYgNQIbkDbaFgF7aDUW7qdWZZZ3sJwTEYUwxC6sR6Y9tRkAJiWwYAXRI6nvbWqH9sSVK0Bpbk9oezwnuIf63BEWBy4AIUuVKmfE3VwdW3UC6L-AbTtB1+Xh4TWlcNukg77x8DMUwRFiZjHNzJgISBhrLk8ZJxLqW6YB++GpBspGIDAMQN-wJf4Y0PbxJEJinHEsQADJb7z0eK9iVpPCU2B1CKLfEebH05KTCeQojiLA+LeQ2s58CJHlowW6P8ywEzRhjcSBoEpS0TATIsxASbADEAABllkKJosAMbt0LmPF+UA35iCEtrX+1C-qdG6oQW6vt-YQn3E4bOCZ-58COJuWAzggE1jIdyC4OlhJpCEftPS9RKxIx9KQzuFCqHn1aj6WhJFs4+31soXM-dS7l0rsQaukxmL10bkDTBrdtFoE2vWas+jlRl2YkYkxdcG5NzrJ+as1i0CdSYX1AaQ1brgKgHUZBHwEp-TgP1Do8tRS3VgSjKxGNI5gEFJo7eu9myYPwJHRkFQeCEGgKYSOaBbpLBWPgexXVeJiHwA0tJKkwzJFumkrJ1tKm0DaTAMAf19y0GID9bOmjVo+y1CAlY2SfTFnetQIU3jamOMei4gGbizEeKBl4qstTfEJiRlDJwnh7hpCSQg0wJMCj0Omb6XOIkMBqHLv3Lp1TFmMHwIxHIFduK8gAHJh1uqeN+XDbk6MXP3e5jzVlVxrhsix4lFx7N9EUW0n1Yn938b1fqg17ghLDmE-M0TYnUHiTAQFHwslwMTIuV5ekeDZGYv3A01SiEwFWgmUZ8jhJOAeYYtZsKGDmKbieM84Nx6mEntPBgs9izzycEDTUHM6pr0PlvDQO8qUJlCZA6BZLNXoMJrONwMBsFcvgajC5SCUFrzNQalJNkJKSmwSjRB+ZwlOqQU66h1DbXqNtZoj5XUHAsIwn7JS7C0ggvUVGoBqKBnF25by6FxiBXECFVsj4SKNGDDQAY5N6zBWbMkriE1f0J5TxnnPXk8rxKKuoMquo0JLjrVVZSg5OjdQcG2g0DFBlaDc2SKJZIgMuXiS7WTaIYoWGOHUGKYUcB8hMNJjtH6KrE7J0Jt29OIL96H07Vuu+D991kzYGfLIaBYYbtTuJNO38uX5Fzf2gOjCPrpk9uU6mkpU4aHpsenamcQUaOzshVCQ1IKYRAAAFkg4gXBpE05UBzGgOqmFUDmkoqxXNYUPkvGQp4Jw5AsLK2Kjy7magmiEH6qqCEnyXF0CwBFdSjHWT6g+FgWI1BfAv0aFAFQdFgiRyitPdo7xI7BGoC8YIBlsMSfopmS4yQqIoWIJcWA8GSJAA" target="_blank"><img src="flems.svg" height="30"> <b>Factored Todo class, on Flems</b></a>
