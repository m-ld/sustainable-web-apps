# Sharing the Todos

Now that we can add Todos and see them, let's consider how to share them. The magic of using a Collaborative Web Library implementing a shared graph is that we don't have to change the writing and reading code to do this.

However, we do have to consider how to distribute the _identity_ of the graph, so others can connect to it. A common practice in sharing apps is to include the "location" of the shared information in the page URL. With conventional cloud apps this is issued to the server, and therefore the identity is in the URL path or search. With local-first apps, it can alternatively be in the fragment, which is not sent to the server.

Here, we'll actually use the search, because setting it forces the page to reload and that reduces the amount of code to write.

Let's change the initialisation of the `config` constant to use the window location search:

```js
const domain = new URLSearchParams(document.location.search).get('domain');
const config = {
  '@id': uuid(),
  '@domain': domain || `${uuid()}.public.gw.m-ld.org`,
  genesis: !domain,
  io: {uri: `https://gw.m-ld.org`}
};
```

Now, we're using the domain specified in the search, if there is one. Notice also that if a domain is specified in the search, `genesis` will now be `false`. This tells **m-ld** that it needs to connect to an existing domain rather than assume it is the creator of the domain.

<aside class="note" title="When using Flems">

Using URL locations is a little awkward with Flems, because we can't see the embedded page URL, and it can't be set in the UI. Let's add some code to use browser prompts to share and set it. At the end of the JavaScript, add this:

```js
document.getElementById('home').addEventListener('click', async () => {
  const newDomain = window.prompt(
    'Domain name (change to switch domain)', config['@domain']);
  if (newDomain && newDomain !== config['@domain']) {
    window.location.assign(`?domain=${newDomain}`);
  }
});
```

Now, clicking the "todos" header will allow you to see and change the domain name. As you can see, changing it works by setting the location search and thus, reloading the page.

</aside>

That's all that's needed to share your Todo list! Let's try it out:

1. Add a Todo or two.
2. Duplicate the browser tab.
3. If you're using Flems, click the header on one tab and copy the domain name, then click the header on the other tab and paste it, then click OK.
4. If you're using localhost, copy the domain name from the console and add `?domain=` and then the domain name to the second tab's URL.

Now you'll see the Todo lists staying in sync across the two tabs. You could, of course, open another browser window somewhere else and do the same, and it would still work.

<a href="https://flems.io/#0=N4IgZglgNgpgziAXAbVAOwIYFsZJAOgAsAXLKEAGhAGMB7NYmBvAHgEIARAeQGEAVAJoAFAKIACEmQB8AHTQtJUMVAxoA5gF4ZIJttloZxBTAwATfYcMscxDGOqEMAJzgxiWkAFdiYALQAObTEAegtiKxs7TBwPADcIGAB3AAdaJ3CQe3pGBg9EiFNiQg1TGHjqGF98wsIKMQg0CGIIDChfOGpWmA0ARiDQuUsjZuJYKT4YLGSVRjFAIgIxPlpTWgBZADUeFmCRscGjYMITc32WACNlgE8wq1dqZvp7FTg4D2Jl2gxk5KCCjy+fiAbuEjEczDAnE8MC8PGDSk49PshlZCD0pCw7H9tIRaDg9O8VnBthh0Yc0UiQVYGslvBTkcRphgKjioPCPAB1RzEMRoGAwUxwMTvMRnGBiFa8gD82jplNs3loYFo1E8cFlQ2oz1e2l5iV8BNoMoMcqGWJAuqN9KGpggcAwZ1gpnVxAGxqG2zhEP0Yh9vpYdweaChMO0WAwDV+pg8YYjQLkvoTYhYniUmuh2pABt8UFtGXqUe0ObgGW9iYTLEYUxmYrNWcrjMYiKDZcTLBz+beHybLZbLBtsWDGfiSW7Pd7KlFSjNE5gUD02xnUFLY79wX7y7HC4gG7L23r1Z3q5Th+2AYg9A3p5g93PaDC2wupmucg9pCXckoIFcsBv9AQiBAABmAAGRBgJAABfCh0GwXAAPwAArBAqDoBgmGIPAICmNJuWATV6BgOpPE8AoILEMAnFxMQAHISAZOBEGCYJ+TUGBELgfAsGzUx8DSNRmIADxdBpSgEzikOogBuOQsNSdIxGAVZJjSS4ABkylnMiKKo2jiHoxjmNMVj2M47jeKcfiYCE4IcCwFTsw0qBxLgKSZOw+TgAASVoAAlZTGDgLTKKwGi6OSBimJYtikNM1lzMs6y4GVABrNx8HPZzXMadzcMSDBiAcABFTwIUuIKdLCiLDOMmKuLivjBJdJwxKwCTpIMNBUOLcVcXDIMNB5JIxAAVR81SAGUTCcBwhGcbA4AAChWFUcAYfAoGVfLb3wVxnAcABKfBWOIBbqJWGM0Go-b2q67lUMgNQxAG4B4xogABApqMQMRiIKBb9ooV7qLe86+q+nqLrEAAfKGxAAAwAEmAX7TH+iD8BpB0IGoI7Elini+LhwHm1Y3k4Ftb62FBhpiZ9c9vuRpwIG+uHKoMtQ8bqgmLLhiC5Agm6-25OzSiUAaMDypongIhbdTEJS7KcNTHLqby-LsgK6nuiA1GuuQutoWB1toNQFrh1ZllnewnBMRhTDEfKxCR7W1GQYHqcugBdCC4b1jrbsGxJPLQGluQG5bPFW4gjrcERYCjgAhS5PNR6jdSu9rdWD0P8BtO0HX5J7yNaVxM6SbPvHwMxTBEWJ0NU3MmAhU7UsuTxkmouoFpgfanqkBTXogMAxG7-BW6ejQBuokQ0KcaixAAMgXwOK+j2JWhK9amDUIpe5e5sfSYpNq6FI5Fg+DcRdnfBEiZxgFv3stdhgb6s5Dyv16gErad9CDDs6AqhAFoGyNhCSiThrq+leofYIiwjhBk1FNU+NZ37cguAJciaRkGDSEvUSs0CV6oPwJ-EqRdqJZR9HzNAf92r6yFmISiABHIueVAHFVKgtK+UA6jFnymKDQ-deGMHwDbMwD8gYg3gNQJmopwbUUlJ9OowNEhHBtuDYA71PrfXkYooUTRYDaMlPPKhf8-YByLGHHqK10Ix2IHHSY6Ek4p1OhYjOdC0DdX3HwouEco62PsYnZOqc6yTAbDANxaAmE7U8GcDoMiYALQNIKARA9mzmNzJ5SsyShQfA4mGZIiSPh91SQmAOFEMBqCjkXLxwjUI5GjvhXkAA5S2iSnAlT9gmcplSbHHQCY4oJLj7SziuvgRgQkeDZHQtUj4Yz9EwHagmG2xBPBOCDN0vxfT44DOcdRA0ETKGdOULmERMBGQVB4IQaApgbZoESaE6sdR8DPIsZkyYcA-Y0MGGgXxvTY7bIYE41OOIcCjOrrXeujdeROFOpqbGyVO4OzgJcTqI9e4pMflkDx3JdQcF6g0FhIlaB42SMFZIJ0CHUTxZDaIYpgGOHUGKYUcB8iAIhn1faiKXZuxBvir2Ryh4j1xXyxey9hWQzYJPLFD0eUe2op7PeBD8g-OJcbAB210w6zuXDSUHsNBI3FX1H2RyTF+0-N+a8gZ-wgAACwAE4wKQU9lQHMaBkr-lQOaWCmEflWXwNQF4n5VnkAAmzJiTgJZHSaIQGJqoIR1JsXQLANluIptZPqD4WBYjUF8OvRoUAVBIWCDbOydd2jvBtsEANcBggiT9dWz8xBLjJDgl+JtsAnUQSAA" target="_blank"><img src="flems.svg" height="30"> <b>Todo list with sharing, on Flems</b></a>