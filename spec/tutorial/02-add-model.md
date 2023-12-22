# Establishing a Model

At the moment, the "Todos" app just shows an input box for adding the first Todo. You might think of the state this is presenting as an empty "Todos UI model" – or, because the box is disabled, a state _prior to_ the availability of a model that can be added to.

In a Sustainable Web App it's important to start thinking of the model early, because it is the unit of editing and data distribution. In due course, we're going to be _sharing_ the Todos model between users as a _linked data graph_. There are other ways to structure the user experience; but for the purposes of this tutorial, we're going to have a simple 1:1 mapping between the UI model and the graph.

So, the first thing we have to do is establish the sharable graph model. This is not something that is provided by browsers yet, so we need a library (we called it the "Collaborative Web Library" [in the specification](http://localhost:3002/web-zero/spec/index.html#reference-architecture)). In this tutorial we're going to use the **m-ld** [JavaScript engine](https://edge.js.m-ld.org/).

To get started we need a JavaScript file. If you're using Flems, just switch to the ".js" tab. Otherwise, create a new empty file next to your HTML file, called `app.js`, and link the file to the HTML by adding a line in the `head` like this:

```html
<head>
  ...
  <script src="app.js" type="module"></script>
</head>
```

In your JavaScript file, we need to import the **m-ld** engine, with a couple of other pieces representing bootstrap choices:
- We don't yet want to persist data in the browser, so we'll use `memory-level` as our persistence backend. (The **m-ld** engine exports this option directly, for convenience.)
- So we don't have to set up our own message delivery service, we'll use the public **m-ld** Gateway, which exposes a Socket.io service, for which we need the client adapter, `IoRemotes`. (Message delivery is called "remotes" in **m-ld** parlance.)

```js
import {clone, uuid} from 'https://edge.js.m-ld.org/ext/index.mjs';
import {MemoryLevel} from 'https://edge.js.m-ld.org/ext/memory-level.mjs';
import {IoRemotes} from 'https://edge.js.m-ld.org/ext/socket.io.mjs';
```

To wire these pieces together, we will call the `clone` function, passing in:
- The persistence backend
- The remotes constructor
- A JavaScript configuration object providing:
  - An identity for the local clone of the graph. This can be any string that is unique; we'll use a **m-ld** utility function to get a UUID.
  - The "domain name", which is an identity for the graph itself. This will be the same for all clones of the graph, or "domain" in **m-ld** parlance. When using the Gateway, this domain name has to follow a specific pattern, again using a UUID.
  - A flag to tell **m-ld** that this is the first clone of a new domain, i.e. "genesis".
  - The Gateway URI for Socket.io to connect to.

```js
const config = {
  '@id': uuid(),
  '@domain': `${uuid()}.public.gw.m-ld.org`,
  genesis: true,
  io: {uri: `https://gw.m-ld.org`}
};
const model = await clone(new MemoryLevel, IoRemotes, config);
console.log(`Model created at ${config['@domain']}`);
```

<a href="https://flems.io/#0=N4IgZglgNgpgziAXAbVAOwIYFsZJAOgAsAXLKEAGhAGMB7NYmBvAHgEIARAeQGEAVAJoAFAKIACEmQB8AHTQtJUMVAxoA5gF4ZIJttloZxBTAwATfYcMscxDGOqEMAJzgxiWkAFdiYALQAObTEAegtiKxs7TBwPADcIGAB3AAdaJ3CQe3pGBg9EiFNiQg1TGHjqGF98wsIKMQg0CGIIDChfOGpWmA0ARiDQuUsjZuJYKT4YLGSVRjFAIgIxPlpTWgBZADUeFmCRscGjYMITc32WACNlgE8wq1dqZvp7FTg4D2Jl2gxk5KCCjy+fiAbuEjEczDAnE8MC8PGDSk49PshlZCD0pCw7H9tIRaDg9O8VnBthh0Yc0UiQVYGslvBTkcRphgKjioPCPAB1RzEMRoGAwUxwMTvMRnGBiFa8gD82jplNs3loYFo1E8cFlQ2oz1e2l5iV8BNoMoMcqGWJAuqN9KGpggcAwZ1gpnVxAGxqG2zhEP0Yh9Ym2dweaDC2wupmucg9pCg+koIFcsHuEHoCEQIB6ABZEAAmHogAC+FHQ2FwqfwACsEFQ6AwmMQ8BApmlucBNfQYHVPJ4CnmxGAnLixAByEgMuCIYLBflqGDluD4LC+Vn4NJqScADxdDVKa-nFcHAG45A3UukxMBVpM0pcADJlGBQHt9gfD4ij8eT0zT2fzxemZdOVcYA3YIcCwK9FzvKBdzgA8j0bU9gAASVoAAlS9GDgR9+ywIcR2SMcJynGcKx-JcV3XF04GVABrNx8CTaDYIMNBqzgblq0gNQxA0M85B9QcAAECkHRAxE7AoAAoAEoKD4ocBJWLAMAaESxAAAwAEmAcTTGkvN8BpB0IGofA1ESUi-xXNTZLQH1p15OBbVE4gnE8ds5KTUTtKcCBRLUvCCOCMyLP-NQ1LzOQ80PFjk25MDSiUHiMESZT2KgNsJN1MQLzApwb0gupkLQsCMLqDiIDUKTotY2hYHwdK1AktTVmWe97CcExGFMMQMG5LTyrUZBBMU5S0EHABdPM1KquRY3jGBE2TPAeizRAAAZ83GqgoAaaiU1Qc1i3rNBt3wagXljTwnHIVMAvfJxktMppCE8M5VQhascmIM7cRA38-tZfUPiwWJqF8WJVGgFQK2CDqwNiSo2LSGBgnOuBgi3ICzouqhiEuZISzjPHYE2vMgA" target="_blank"><img src="flems.svg" height="30"> <b>Created model, on Flems</b></a>