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

First, we have to consider how to respond to updates. The **m-ld** engine, as represented by our `model` variable, offers a [follow method](https://js.m-ld.org/interfaces/meldstatemachine.html#follow), with which we can receive [**m-ld** update](https://js.m-ld.org/interfaces/meldupdate.html) objects containing fine-grained deletes and insertions into the graph. Since we know the behaviour of our app – the only thing it does is add new titled Todos – we could just inspect the `@insert` component and add the titles to the UI. However:

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
  "@describe": "?id", "@where": { "@id": "?id", title: "?" }
}));
```

The function takes the model we want to query, and a procedure to run. This procedure is passed a `state` object representing the current state of the domain graph, which we can query using the `read` method to obtain, in our case, a set of Todo objects. The query we're using is a ["describe"](https://edge.js.m-ld.org/interfaces/describe.html), which is a convenient way to obtain all the properties of matching objects.

Finally, let's `subscribe` to the reactive Observable and use the template to replace the contents of the Todo list using the new array of Todo objects provided by the query:

```js
const list = document.getElementById("list");
const template = document.getElementById("todo-template");
roq.subscribe(todos => {
  const listItems = todos.map(todo => {
    const fragment = template.content.cloneNode(true);
    fragment.getElementById("label").textContent = todo.title;
    return fragment.getElementById("todo");
  });
  list.replaceChildren(template, ...listItems);
});
```

The magic of `watchQuery` is that we only have to provide a query, not any update handling; but still, for every subscription notification, we'll get the results of the query – as if it is being re-run on every update.

<a href="https://flems.io/#0=N4IgZglgNgpgziAXAbVAOwIYFsZJAOgAsAXLKEAGhAGMB7NYmBvAHgEIARAeQGEAVAJoAFAKIACEmQB8AHTQtJUMVAxoA5gF4ZIJttloZxBTAwATfYcMscxDGOqEMAJzgxiWkAFdiYALQAObTEAegtiKxs7TBwPADcIGAB3AAdaJ3CQe3pGBg9EiFNiQg1TGHjqGF98wsIKMQg0CGIIDChfOGpWmA0ARiDQuUsjZuJYKT4YLGSVRjFAIgIxPlpTWgBZADUeFmCRscGjYMITc32WACNlgE8wq1dqZvp7FTg4D2Jl2gxk5KCCjy+fiAbuEjEczDAnE8MC8PGDSk49PshlZCD0pCw7IQnDAwB58Hp3is4NsMOjDmikSCrA1kt5KcjiNMMBVCLQoPCPAB1RzEMRoGAwUxwMTvMRnGBiFb8gD82npVNs3loYFo1E8cHlQ2oz1e2n5iV8hNocoMCqGfz1SRNDKGpggcAwZ1gpk1xAGpqG2zhEP0Yj9-pYdweaChMO0WAwDV+pg8EajQLk-qTYhYniU2uhupARt8UHtGXqMe0ebgGV9yaTLEYUxmEot2Y+hsmTMYiJDFeTLDzhbeHzbHY7LDtsVDWfiVoT7YHFa7jpgSnrKnFUD02yX8-L04DwWHm+na4ge5nO2btaP27T5+2QYg9D315g91vaDC2wupmuci9pCg+koIFcWAn3oBBEBAHp-EQABmXwIMQHoACYQAAXwodBsFwMD8AAKwQKg6AYJhiDwCApjSXlgG1egYDqTxPAKZCxDAJxaCwMQAHISEZOBEGCYJBTUGAcLgfAsFzUx8DSNR+IADzdBpShk0TcPYgBuORSNSdIxGAVZJjSS4ABkynnRjmNYjiuOSHi+IEoTcNE8TJKcaSYDk4IcCwAzcxMqBlLgNSNLI7TgAASVoAAlfTGDgMyWLYzjiG43j+NMQThMc9lnNc9y4FVABrNx8FvfzAsaYKKMSDBiAcABFTwIUuOKLMS5LbLS+yRLErKpNkt0nCUrAVPUgw0AI0ssjQSA1DEDQdMTMRtAAAQKbREDEOiCgACgASgoBblpWOMDCQMQAAMABJgE20xduQ-BaSdCBqHwNREkyiSpLO-b20E-k4HtdbiCcBqfr9W91uupwIHWs6rJs4I3o+7KzuQuRkJG8beS80olDmjAqqaJ5qK2-UxD0rynCM3y6nCqKvJiuoCOmnbMZAtkhKgWg1C2s7VmWed7GxarBTEaqxCu5mIDUZB2KWo7IzQdiAF1kLO1nBjGkDeX1UK0FpXk5pWNUcAYV63BEWBTeIAAhS5Qtuy1Em0DW0F1-XvHwO0HSdUW5rAVpXBGuR3YN-AzFMERYiIwz8yYCEtu0QrLk8QE6i2mAdtmqR5vbCAwDEDP8GT2aNDm7QREIhFMgAMhrvkkj1sPYlaBr8FgdQiiz4AFr9HH53wRJocYLae6nf1dhgdbQ89luoFB3uxGQnb8E6GrCC28aOfwCEWKcVmB0XvjFiOENtRMSEijrD3eQuGSmLSEUjgbuT6mrReZ+IfA54a2bFpAbQI0-RozQMvYOWs0ATRYgARz-lVde9VGpbX7lAOopYRbZzEOgxg+Bha3THn6Q68BqDQ3FGtf+0pVqUH-ktRIRxsTkOADQqh61tCUJdNQyerCQCykyCA5ers5BY2UPmP+xtPDW3NsQS2kwiJ2wdonEAJYMiu2EdWFsEojaqgkURKRMjrbyMdg2FYTYawixdiNGB+A4CeDOB0UhMAtpGmFBoHOBDJoTWUaFasLiRQfC6l8JxHxMHuL9MI5iGA1DWz-uo2sq9si6KovyAAcgLJxINM5AP9BEqJujBLSKtnI+2Rj1wrhACvRgckeAJIYDEj4+BJ5ZL9NiYgngnAhhyZI-J+iikKO0EaCxC0wELWUbgmATIKg8EINAUw2I0BONPCLOo+AVleJ8a7MB-5AKPmDKBEAUEeiIAAAwoWVlQPMaB8qgVQCAaImEQAKTcqvF4-42nkDAvDFKTgCavSaIQWx6oIQERyF-OgWAPLiQheyQ0HwsCxGoL4FujQoAqFwsEbEXlo7tHeNiYI1AXjBEeUpfFeFsyXGSPc0slxYCnOQkAA" target="_blank"><img src="flems.svg" height="30"> <b>Todo list update handling, on Flems</b></a>