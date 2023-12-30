# Introduction

**m-ld** [[M-LD]] is software for live information sharing. It has the characteristics required for a Shared-Graph Library as described in the [Specification](../index.html#reference-architecture):

- The **m-ld** Javascript _engine_ can be included in a web application.
- Data is represented in an RDF graph [[RDF11-CONCEPTS]], with a programming interface based on JSON-LD [[JSON-LD11]] and json-rql [[JSON-RQL]].
- Graphs can be synchronised using a choice of network protocols. The simplest choice is to use the [m-ld Gateway](https://gw.m-ld.org) as a Message Delivery service. A free cloud instance is available.
- If desired, the Gateway also provides a Backup service, which exposes the data in the Graph as Linked Data [[LINKED-DATA]] via its own API.

The TodoMVC initiative [[TODOMVC]] is "a project which offers the same Todo application implemented using MV* concepts in most of the popular JavaScript MV* frameworks of today."

In this tutorial we will implement a Todo application in vanilla JavaScript, starting with some minimal CSS and HTML based on the TodoMVC project assets.

We will assume you have a working knowledge of HTML and JavaScript fundamentals.