# Setup

There are two recommended ways to participate in this tutorial, as follows. Both options start with a little HTML, and link to a provided CSS file for styling.

## Option 1: Using Flems

> Flems is a playground for web development. It's ideal for prototyping ideas & sharing working front-end code examples.

Using Flems will allow you to quickly apply the code changes and see the result, without the overhead of editing files on your computer and refreshing a web page.

The following link starts a Flems environment in a new tab, with the required setup for the following tutorial steps:

<a href="https://flems.io/#0=N4IgzgpgNhDGAuEAmIBcIB0ALeBbKIANCAGYCWMYaA2qAHYCGuEamO+RIsA9nYn6wA8AQgAiAeQDCAFQCaABQCiAAnZQAfAB06gtcqgM6AcwC8mkBDrmtV+LogMkNzfBeDm8BsthYGAJ0h4MxAAV3gSAFoADnNlAHpnVzsPL0ZmYIA3MggAdwAHbj9XEG9efiDzHLIkeCwTJAgs2AgIqpqsQmUyOjJ4MgYoCLBYAYgTAEZYhO0XNz74GHVpCFw8g0RlQCICZWluJG4AWQA1SUE4+cWZuzisBycrwQAjPYBPRLdIBDJebwMwMGC8D23AYeTysWqwVB4JA7yS9kcED8vwY-2Ct0RfmsV1m8Kw43Ugi8WD8EBIwQw1iB+zAZwYhJuBJxSTc3TyYWZuJcawYzSw3CgDT8wQA6r54Mo6BBkGBlEDlI8IMp9lKAPzmTkszxhbgkbiwEJgTW42B-AHmKU5CLU7ga2xcpKQi25O0O3FIMhgBiPGBIY3wab2+E3O5ImzKCPKM6fPq8RJnZ5IN7aM5qGycSAwL68KjoAAMqHGAFYQABfQj0JgsdAYABWVGIPD4lngrDLAF1iFBugBrXO0EBpasgboNAAeGFg-04IT8BHQOHgeTAqDicT8DByGCMvSwIUehqRTfKk+4uDiuAigovV6Q1uBuAysAiGUMFAM9fXK24GRaYCBpJxFOYBxKOEATsBnDwC8eTDv+LwwB2pZAA" target="_blank"><img src="flems.svg" height="30"> <b>Start the tutorial on Flems</b></a>

## Option 2: Using Localhost

Using your computer to host the web application based on local files will allow you to easily keep track of your progress and version control any further experiments. You will need to be able to create and edit text files, and a web browser with an internet connection.

To begin, create a folder with a file called `index.html`, with the following content:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="utf-8"/>
    <meta name="viewport" content="width=device-width, initial-scale=1"/>
    <title>Template • TodoMVC</title>
    <link rel="stylesheet" href="https://m-ld-todomvc-vanillajs.vercel.app/css/index.css"/>
  </head>
  <body>
    <section class="todoapp" id="app">
      <header class="header">
        <h1><a href=".">todos</a></h1>
        <input
            placeholder="What needs to be done?"
            autofocus
            class="new-todo"
            id="new"
            disabled
        />
      </header>
    </section>
  </body>
</html>
```

There are various tools available to serve the page locally so that it can be viewed in a web browser:

- If you have NodeJS installed, you can use `npx http-server` in the folder.
- If you have Python 3, you can run `python -m http.server` in the folder.
- If you have an Integrated Development Environment (IDE), it will typically have an action to serve the file – consult the IDE documentation.