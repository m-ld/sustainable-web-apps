# Sharing the Todos

Now that we can add Todos and see them, let's consider how to share them. The magic of using a Collaborative Web Library implementing a shared graph is that we don't have to change the writing and reading code to do this.

However, we do have to consider how to distribute the _identity_ of the graph, so others can connect to it. A common practice in sharing apps is to include the "location" of the shared information in the page URL. With conventional cloud apps this is issued to the server, and therefore the identity is in the URL path or search. With local-first apps, it can alternatively be in the fragment, which is not sent to the server.

Here, we'll actually use the search, because setting it forces the page to reload and that reduces the amount of code to write.

Let's change the initialisation of the `config` constant to use the window location search:

```js
const domain = new URLSearchParams(document.location.search).get("domain");
const config = {
  "@id": uuid(),
  "@domain": domain || `${uuid()}.public.gw.m-ld.org`,
  genesis: !domain,
  io: {uri: `https://gw.m-ld.org`}
};
```

Now, we're using the domain specified in the search, if there is one. Notice also that if a domain is specified in the search, `genesis` will now be `false`. This tells **m-ld** that it needs to connect to an existing domain rather than assume it is the creator of the domain.

<aside class="note">

Using URL locations is a little awkward with Flems, because we can't see the embedded page URL, and it can't be set in the UI. Let's add some code to use browser prompts to share and set it. At the end of the JavaScript, add this:

```js
document.getElementById("home").addEventListener("click", async () => {
  const newDomain = window.prompt(
    "Domain name (change to switch domain)", config["@domain"]);
  if (newDomain && newDomain !== config["@domain"]) {
    window.location.assign(`?domain=${newDomain}`);
  }
});
```

Now, clicking the "todos" header will allow you to see and change the domain name. As you can see, changing it works by setting the location search and thus, reloading the page.

</aside>

That's all that's needed to share your Todo list! Let's try it out:

1. Add a Todo or two.
2. Duplicate the browser tab
3. If you're using Flems, click the header on one tab and copy the domain name, then click the header on the other tab and paste it, then click OK.
4. If you're using localhost, copy the domain name from the console and add `?domain=` and then the domain name to the second tab's URL.

Now you'll see the Todo lists staying in sync across the two tabs. You could, of course, open another browser window somewhere else and do the same, and it would still work.

<a href="https://flems.io/#0=N4IgZglgNgpgziAXAbVAOwIYFsZJAOgAsAXLKEAGhAGMB7NYmBvAHgEIARAeQGEAVAJoAFAKIACEmQB8AHTQtJUMVAxoA5gF4ZIJttloZxBTAwATfYcMscxDGOqEMAJzgxiWkAFdiYALQAObTEAegtiKxs7TBwPADcIGAB3AAdaJ3CQe3pGBg9EiFNiQg1TGHjqGF98wsIKMQg0CGIIDChfOGpWmA0ARiDQuUsjZuJYKT4YLGSVRjFAIgIxPlpTWgBZADUeFmCRscGjYMITc32WACNlgE8wq1dqZvp7FTg4D2Jl2gxk5KCCjy+fiAbuEjEczDAnE8MC8PGDSk49PshlZCD0pCw7H9tIRaDg9O8VnBthh0Yc0UiQVYGslvBTkcRphgKjioPCPAB1RzEMRoGAwUxwMTvMRnGBiFa8gD82jplNs3loYFo1E8cFlQ2oz1e2l5iV8BNoMoMcqGWJAuqN9KGpggcAwZ1gpnVxAGxqG2zhEP0Yh9vpYdweaChMO0WAwDV+pg8YYjQLkvoTYhYniUmuh2pABt8UFtGXqUe0ObgGW9iYTLEYUxmYrNWcrjMYiKDZcTLBz+beHybLZbLBtsWDGfiSW7Pd7KlFSjNE5gUD02xnUFLY79wX7y7HC4gG7L23r1Z3q5Th+2AYg9A3p5g93PaDC2wupmucg9pCXckoIFcsBv9AQiBAHoAAZEAAZgAVhAABfCh0GwXAAPwAArBAqDoBgmGIPAICmNJuWATV6BgOpPE8AooLEMAnFxMQAHISAZOBEGCYJ+TUGBkLgfAsGzUx8DSNRWIADxdBpSiE7iUNogBuOQcNSdIxGAVZJjSS4ABkylnCiqJo+jiEY5jWNMdjOO43j+KcQSYBE4IcCwNTsy0qBJLgGS5NwxTgAASVoAAlVTGDgHTqKwOiGOSJiWLYjiUPM1lLOs2y4GVABrNx8HPVz3MaTz8MSDBiAcABFTwIUuEK9IiqLjNMuKeISgThJdJwJKwKTZIMNB0OLcVcXDIMNB5JIxAAVT89SAGUTCcBwhGcbA4AAChWFUcAYfAoGVQrb3wVxnAcABKfB2OIJbtBWGMDBAQ7Op67l0MgNQxCG4B4zEbQAAECm0RAxFIgolsOih3q+y6Bt+vqrrEAAfGGxAAAwAEmAAHTCBqD8BpB0IGoE7EniviBIRkHm3Y3k4FtP62HBhpSZ9c8-tRpwID+hHqqMtQCYaomrIRqC5Cgu6-25BzSiUIaMAKponiIpbdTEFSHKcDTnLqXyAocoK6keiA1FuuQetoWBNtoNQloR1ZllnewnBMRhTDEQqxBR3W1GQWjPtptBaIAXSghGDa6+7hsSby0BpbkhtWzx1uIE63BEWA44AIUubz0Z1Ecbs63Vw8j-AbTtB1+ReyjWlcTq5DziPvHwMxTBEWJMPU3MmAhc6QHSy5PEBOolpgQ6XqkJT3ogMAxAH-Bu5ejQhu0EQMIRTIADIV9D-O69iVoys2pg1CKIe3ubH0xdnfBEhZxgluPstdhgP6a4L7eoDK+nfSg47OiKwglqNk2ITUScLdHs70fQsUWEcIMmoZpCiOPUWu3ILhCUomkOBYpeQiXqJWMBG9EH4BfmVMuMoQCdR9ALNAn8q7dRFmIaiABHMuBUf6lXKktM+UA6jFkKmKDQI9uGMHwHbMwN9QYgC9vAagLNRSQ20JKH6lAPriMSEcO2kNgBKO+k6JASj5HaLqPfP6ciggUM-kHQ2tCixRz6mtTCCdiBJ0mJhNOGdO5WO0EHEO+4eFlxjnHexjjU7p0zpmD4+pJgNhgB4zqDC9qeDOB0aRMAloGkFHw0ezYQ5WO8pWNJQoPhcTDMkFJHxh4ZITCHKiGA1BxzLt4wR6Ecjx0IryAActbFJTgypBwTFUmpdjTqBOccEtx9pZwePwIwESPBsiYTqR8SZTRYBkN9HbYgngnBBj6f4wZydhmuO0AaaJ70qHvSsUImAjIKg8EINAUwds0ApIidWOo+A3nZNyUHU5XU-EDMTnshgLiQk4jxDdeuphG7NwYK3Ys7cnCd01LjVK2g6jQkuN1SeQ90m3yyGgXquoOD9QaEwsStACbJFCskM6uDtCEuhtEMUf9HDqDFMKOA+Qf5QwGodFFuKnrIDBkS66vsen1AnvLJIdKBpiDXqHKVxK2Bzz5XrAV4jvbaBFeU30+Q0ArAJltb+u10x60eQjSU3sNAowJUKgOorTFB0-N+a8gZ-wgAAJyIB6G66CvsqA5jQKlf8qBzTwWwrqmy+BqAvE-Bs8gAEOYsScFLE6TRCDxNVBCRpdi6BYDsrxPNrJ9QfCwLEagvht6NCgCoFCwQ7YOWbu0d4dtghRrgMEMSEbW2fmIJcZICEvw9tgD6qCQA" target="_blank"><img src="flems.svg" height="30"> <b>Todo list with sharing, on Flems</b></a>