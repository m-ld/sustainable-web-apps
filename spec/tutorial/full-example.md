# Complete App Example

In the above steps, we have implemented a simple example app based on TodoMVC, which demonstrates the main concepts of the Shared-Graph Model Pattern. The app Model is a **replicated semantic graph**, stored locally and shared with other instances of the app.

The project https://github.com/m-ld/m-ld-todomvc-vanillajs showcases a completed app implementing the full TodoMVC specification, including task statuses (complete and active), and filters. It follows the same pattern as this tutorial.

Draft pull requests in the project show further refinements of the collaboration behaviour and security, as follows:

- <a href="https://github.com/m-ld/m-ld-todomvc-vanillajs/pull/2" target="_blank">Use a text CRDT for collaborative text editing</a>
- <a href="https://github.com/m-ld/m-ld-todomvc-vanillajs/pull/4" target="_blank">Using KeyCloak for application login</a>

Finally, another pull request demonstrates <a href="https://github.com/m-ld/m-ld-todomvc-vanillajs/pull/5" target="_blank">using React</a>, as an example of how to adapt the vanilla JavaScript approach to a JavaScript framework or library.