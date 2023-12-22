# Adding the Todo List UI

Now that we can add Todo items to our model, we need to show them in the UI. As we mentioned, the Collaborative Web Library will provide us with _updates_ when data changes; and we should respond to all updates through the same code path whether they come from the local user or from a remote one. 

In general, handling an update and applying it to the local HTML is a matter of translating it into a new state of the HTML Document Object Model (DOM). Web frameworks and libraries like React are great for handling this concisely and reliably; and as you scale up your app ambitions it would be a good idea to use one.

We'll keep using plain JavaScript for now. For simplicity, we'll just refresh the whole list on every update; when using a framework with DOM diffing, this approach magically becomes efficient and will scale to large lists.

Add the following HTML, just after the existing <code class="hljs-name">header</code> section, and within the outer <code class="hljs-name">section</code>:

```html
      <section class="main" id="main">
        <ul class="todo-list" id="list">
          <template id="todo-template">
            <li id="todo">
              <div class="view">
                <label id="label"></label>
              </div>
            </li>
          </template>
        </ul>
      </section>
```

This HTML markup establishes an identified space in the DOM for the Todo list itself, and a <code class="hljs-name">template</code> for each Todo item, which we'll use to add items to the list dynamically.

First, we have to consider how to respond to updates. The **m-ld** engine, as represented by our `model` variable, offers a [follow method](https://edge.js.m-ld.org/interfaces/meldstatemachine.html#follow), with which we can receive [**m-ld** update](https://edge.js.m-ld.org/interfaces/meldupdate.html) objects containing fine-grained deletes and insertions into the graph. Since we know the behaviour of our app – the only thing it does is add new titled Todos – we could just inspect the `@insert` component and add the titles to the UI. However:

- If we are connecting to an existing Todo list (as we will shortly), we expect to find Todos already in the graph. They won't appear as insertion _updates_.
- If Todos are removed, or their titles changed, then listening specifically for insertions will not capture the actual changes.

Directly "following" updates therefore involves the complexity of querying for an initial state, with which we populate the initial UI, and then translating individual patches into UI changes. This can be considered a "low-level" API, and it is provided by **m-ld** for all general cases. However, for our specific case, not only is it complex, it is also contrary to the way libraries like React work. They expect to do the "diff" part themselves, when presented with some new state.

Fortunately, the **m-ld** engine also provides some higher-level utilities targeting our use-case. Specifically, we can use "Reactive Observable Queries" (ROQs) to inspect some state, and be delivered _new states_ when changes happen, instead of _updates_. ROQs are found in an extension that targets a "reactive" approach using RxJS [[RXJS]]. Let's add an import for the [library function](https://edge.js.m-ld.org/globals.html#watchquery) we'll use:

```js
import {watchQuery} from 'https://edge.js.m-ld.org/ext/rx.mjs';
```

Down at the bottom of the JavaScript, let's use the function `watchQuery` to get a reactive Observable, to which we can subscribe:

```js
const roq = watchQuery(model, state => state.read({
  '@describe': '?id', '@where': { '@id': '?id', title: '?' }
}));
```

The function takes the model we want to query, and a procedure to run. This procedure is passed a `state` object representing the current state of the domain graph, which we can query using the `read` method to obtain, in our case, a set of Todo objects. The query we're using is a "Describe", which is a convenient way to obtain all the properties of objects matching the given _pattern_, in which something beginning with a question mark is a _variable_. You can learn more in the [Describe documentation](https://edge.js.m-ld.org/interfaces/describe.html).

Finally, let's `subscribe` to the reactive Observable and use the template to replace the contents of the Todo list using the new array of Todo objects provided by the query:

```js
const list = document.getElementById('list');
const template = document.getElementById('todo-template');
roq.subscribe(todos => {
  const listItems = todos.map(todo => {
    const fragment = template.content.cloneNode(true);
    fragment.getElementById('label').textContent = todo.title;
    return fragment.getElementById('todo');
  });
  list.replaceChildren(template, ...listItems);
});
```

The magic of `watchQuery` is that we only have to provide a query, not any update handling; but still, for every subscription notification, we'll get the results of the query – as if it is being re-run on every update.

<a href="https://flems.io/#0=N4IgZglgNgpgziAXAbVAOwIYFsZJAOgAsAXLKEAGhAGMB7NYmBvAHgEIARAeQGEAVAJoAFAKIACEmQB8AHTQtJUMVAxoA5gF4ZIJttloZxBTAwATfYcMscxDGOqEMAJzgxiWkAFdiYALQAObTEAegtiKxs7TBwPADcIGAB3AAdaJ3CQe3pGBg9EiFNiQg1TGHjqGF98wsIKMQg0CGIIDChfOGpWmA0ARiDQuUsjZuJYKT4YLGSVRjFAIgIxPlpTWgBZADUeFmCRscGjYMITc32WACNlgE8wq1dqZvp7FTg4D2Jl2gxk5KCCjy+fiAbuEjEczDAnE8MC8PGDSk49PshlZCD0pCw7H9tIRaDg9O8VnBthh0Yc0UiQVYGslvBTkcRphgKjioPCPAB1RzEMRoGAwUxwMTvMRnGBiFa8gD82jplNs3loYFo1E8cFlQ2oz1e2l5iV8BNoMoMcqGWJAuqN9KGpggcAwZ1gpnVxAGxqG2zhEP0Yh9vpYdweaChMO0WAwDV+pg8YYjQLkvoTYhYniUmuh2pABt8UFtGXqUe0ObgGW9iYTLEYUxmYrNWcrjMYiKDZcTLBz+beHybLZbLBtsWDGfiSW7Pd7KlFSjNE5gUD02xnUFLY79wX7y7HC4gG7L23r1Z3q5Th+2AYg9A3p5g93PaDC2wupmucg9pCXckoIFcsBv9AQiBAABmAAGRBgJAABfCh0GwXAAPwAArBAqDoBgmGIPAICmNJuWATV6BgOpPE8AoILEMAnFxMQAHISAZOBEGCYJ+TUGBELgfAsGzUx8DSNRmIADxdBpSgEzikOogBuOQsNSdIxGAVZJjSS4ABkylnMiKKo2jiHoxjmNMVj2M47jeKcfiYCE4IcCwFTsw0qBxLgKSZOw+TgAASVoAAlZTGDgLTKKwGi6OSBimJYtikNM1lzMs6y4GVABrNx8HPZzXMadzcMSDBiAcABFTwIUuIKdLCiLDOMmKuLivjBJdJwxKwCTpIMNBUOLLI0EgNQxA0BT4xogABApqMQMRiIKAAKABKChhuokaVhjNAJrEAADAASYBptMeaIPwGkHQgah8DURJYp4vjNsW5tWN5OBbUm4gnBK+6fXPSa9qcCBJs2yqDMu674s2iC5Ag9quu5OzSiUQaMDypongImbdTEJS7KcNTHLqby-LsgK6lQvq5uhv9aFgfAoFoNQZs21ZllnewnBMRhTDEfKxF20mIDUZBltW8N1oAXQgzbycGTq-25XVPLQGluUGlYVRwBgLrcERYHV4gACFLk8g7qN1aipbQeXFe8fAbTtB1+QG8jWlcdrLaV-AzFMERYnQ1TcyYCEZuo1LLk8ZJqLqGaYDmgapCG5sIDAMQo-wEOBo0QbqJENCnGosQADJ855JIFfd2JWhKmmmDUIoY+AYafSYpNPaFI5Fg+Dc4dnfBEj+xgZvr5sE12GBJrd63y6gD6G7ECC5vwToCsIGauqptiIUopxyd9Gem74I4g01ExISKGsre5C4BPItJW7FXkhPqSsZ-H4h8EnkrHeorKfQhtA5-auQMMxCUQAI6OzykvYqpUZpdygHUYs+UxQaDjggxg+A2ZmAHktFa8BqB-VFBtaikpxp1GWokI4bMNrAFGuNSaRCSFCiaLAOhko86-znubIBRZlbimVJ4XWmtiDa0mOhA2Rsg7cLNhTNA3V9yIMdqrfh6FBHCN1mI42dZJgNhgFIuQoD8BwE8GcDo+CYAzQNIKZB8cfRcNzJ5SslihQfA4mGZI5iPix2sb6IBFEMBqF1o7ORaDUI5FfvhXkAA5Zm5j3rR3agmXx-jlGsSETrURhtjaLjNvgRgQkeDZHQoEj4OSmEwHib6NmxBPBOCDIkgRKTVHpPEdRA0ujmz-2Gtw9BMBGQVB4IQaApg2ZoHMVo6sdR8CTO4fYyYcBzYdLQJ+b815Az-iAj0RAABWAATJBUWVAcxoGSv+VA5pYKYTQKJBeLxPzVPIABIGTEnBIwuk0QgRjVQQhCcougWAbLcX+ayfUHwsCxGoL4cujQoAqCQsENmdkfbtHeGzYI1AXjBBElZa5yFMyXGSHBL8xBLiwD2RBIAA" target="_blank"><img src="flems.svg" height="30"> <b>Todo list update handling, on Flems</b></a>