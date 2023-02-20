# Web Zero Demo

> It is better to have 100 functions operate on one data structure than 10 functions on 10 data structures.  
> – Alan Perlis, “Epigrams on Programming”
## Background
There is a common (though not at all rigorous) distinction made between the “Web 2.0” era of the World Wide Web and the earlier “Web 1.0” era. Notably, both kinds of sites coexist on the actual current Web: the terms refer to approaches to the architecture and use of certain technologies for individual websites. For the sake of this document, consider the following definitions:

### Web 1.0
In Web 1.0, individual pages are stored on the server as static HTML files. Pages are created by uploading new files to the server, and pages are edited by uploading new versions of existing files to the server. Examples include [the first website](http://info.cern.ch/hypertext/WWW/TheProject.html) and [example.com](https://example.com/).

### Web 1.1
In Web 1.1 (which is an invention of this document and not a common term), dynamic processes on the server synthesize HTML on the fly from data stored in some kind of database. These sites are implemented in languages like Perl and PHP. Examples include the classic Yahoo, a directory of the Web.

### Web 2.0
In Web 2.0, content is primarily authored by users. The website (now a “web app”) is generally a platform hosting user content. The hosted data is not intrinsically portable—it must be exposed with a separate API. Examples include [Facebook](https://www.facebook.com/) and [Reddit](https://www.reddit.com/).

## The Problem
In each of these kinds of websites, how does a piece of content evolve? In Web 1.0, you edit HTML offline, then upload it to server. In Web 1.1, you connect to the database and make changes. In Web 2.0, you use interfaces provided by the web app itself to make and publish changes.

These solutions depend increasingly on the availability of the service—first the database, and then the web app. Only the first solution, Web 1.0’s, allows for completely offline and non-server-mediated editing, relying on the server only to accept the uploaded HTML as the new published version at the end.

This is analogous to moving from a functional approach to an object-oriented approach. In Web 1.0, the entirety of the data *is* the HTML, which is carried everywhere and can be edited anywhere. Anyone with an HTML editor (which can be any text editor) can edit the HTML of anyone’s website, because editing does not affect *state*, it simply produces a new version. The content itself is *persistent*, in the vocabulary of persistent data structures. The state change (the “mutation”) happens at the end: the upload. Only this operation is (and only this operation needs to be) protected by authorization rules.

In Web 2.0, the data is transmitted in proprietary formats, and perhaps fetched at various times by the JavaScript running in the client side of the app. The client is akin to an *object* which holds its own state and communicates with another object (the server) by sending messages. There is no way to create and modify a copy of the system offline without access to (and without running) the server-side application.

While Web 2.0 has certainly brought some major advances, we feel that this aspect—tying the page to a live, running instance of a backend available for message passing—is a step backwards for collaboration. Each Web 2.0 application must provide its own tools for collaboration within its own system, which means that many applications are less collaborative than they could be (because building the tools would be prohibitively expensive), and users cannot bring their own favored tools but are forced to use those provided by each specific application.

Simply editing the HTML of a page is also semantically difficult, because the data is *denormalized*: the “same” piece of data may appear in multiple places. Consider a blog post in which the author’s name appears on a byline at the top of the article and in a short bio at the end. Editing the name in one place in the HTML does not affect it in the other, so the page can become inconsistent. In Web 1.0, this was how the pages were actually authored, and inconsistencies were common. In Web 1.1 and Web 2.0, the name is stored in a single place in the server’s database, and editing the name in the database causes both page locations to render the new value. But this again depends on a data store that lives outside the context of the page, one that can’t be copied and edited at whim.

## The Solution
We therefore propose what we will call “Web Zero”, which both supports the advanced interactivity of Web 2.0 *and* restores the persistence and portability of Web 1.0’s HTML.

### Web Zero
In Web Zero, pages consist of both the semantic data behind the presentation and the rules to transform them to (and from) the presentation, all of which is transferred to the client. Thus, as in Web 1.0, the client has a complete unit of content which can taken offline and edited elsewhere. And as in Web 1.0, the only operation which must be protected with authorization is the server-side state change in which a new version is “pushed” to become the authoritative version. But unlike Web 1.0, the data that is edited is semantic and normalized, reducing inconsistencies.

Because the data and presentation are portable, and because the data is normalized, various modes of interaction and collaboration become possible. Experimentation is also easier: where today you can open the DevTools and poke at the DOM, under Web Zero you can just as easily poke at the data and see the result.

New versions can then be pushed back to the original by various means, as authorized by the owner of the page, such as HTTP PUTting the new version, offering the equivalent of a pull request, or participating in a m-ld domain of which the original is a clone.

## m-ld’s Role in Web Zero

A Web Zero page will consist of RDF triples (or of data in equivalent formats). Thus, we propose that m-ld is an extremely useful tool for collaborating on Web Zero content. It is by no means the *only* tool: it is one of “100 functions [which] operate on one data structure.” Different modes of interaction can be provided by different tools: git-like history is offered by [TerminusDB](https://terminusdb.com/), flexible querying is offered by [Comunica](https://comunica.dev/), and realtime multiplayer editing is offered by m-ld. This requires no extra effort on the part of these projects. These are simply the tools’ existing RDF capabilities applied to Web Zero content.

In this project, we aim to demonstrate three things:
1. That Web Zero as described here is a sensical concept when made concrete,
2. That it is a useful direction for the Web generally, and
3. That the realtime multiplayer editing of Web Zero pages which m-ld can offer presents a novel and valuable pattern for Web content collaboration.

## The Demo

We will demonstrate a page which carries both presentational information and semantic information. The page will be a blog post, written by an author, who has a name. The author’s name will appear in multiple places on the page—thus, the value is denormalized in the presentational view. When the user uses the browser’s DevTools to change the author’s name in one location in the DOM, the other location will update to match.

Next, we will offer a data view which illustrates the semantic data underlying the page, rendered as N3 triples or as JSON-LD. When the name is changed as above, the change will be visible here as well.

Last, we will use m-ld to synchronize multiple users’ views of the same page. When the name is changed as above, it will also be immediately visible to all other users in the system.
