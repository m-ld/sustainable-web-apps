# Adding Todo List Items

Now that we have a model, we can enable the input box. After the call to `clone`, add these lines:

```js
const newInput = document.getElementById("new");
newInput.disabled = false;
```

Now, you should be able to type into the input box. We want the new Todo item to be added to our Todo list when the user presses \<Enter\>. We do that by adding an event listener for the `keyup` event, like this:

```js
newInput.addEventListener("keyup", (e) => {
    if (e.key === "Enter" && newInput.value.length) {
        // <add the Todo>
        
        // Then clear the input box for the next item
        newInput.value = "";
    }
});
```

How do we add the Todo? At this point it's tempting to start constructing HTML for the list and its items, to go under our input box on the page. But, it's important that we drive the view from the model, not the other way around. This is always true in a well-structured user interface app; in a Sustainable Web App it's even more important, because (in due course) it's always possible that updates will be appearing _from other users_ ("remote" users). The library we use for the model is designed to correctly apply remote updates, in such a way that everyone ends up with the same state – their local states "converge". Convergence is a surprisingly difficult property to guarantee – one of the reasons we are using a library in the first place.

So, the correct pattern is to make changes to the model; then have the changes be "echoed" back from the model. This ensures that the view (the HTML) doesn't get out-of-sync with the contents of the model, regardless of who made the changes and when. Writing to the model also gives the Collaborative Web Library the opportunity to deliver the change to other clones.

![reactive updates sequence diagram](img/reactive-updates.seq.svg "Correct handling of local changes")

We'll get to steps 4 & 5 in a moment, but for now let's code up writing to the model (step 2). In **m-ld**, the API accepts [JSON data](https://spec.m-ld.org/#transactions). For now, all we have in a Todo item is its title, so writing to the model will look like this (to replace the "\<add the Todo\>" comment):

```js
    model.write({
      title: newInput.value,
    }).catch(console.error);
```

We haven't yet implemented any sync with the UI, but there is an easy way to see what's in our model, using **m-ld**'s _playground_. You may have seen that we are logging the domain name to the console as soon as the `clone` function returns. Put the domain name (e.g. "`clq5gi4zo00013b6brnxkya68.public.gw.m-ld.org`") into the box below, and click the link:

<a id="playground-link" href="https://edge.m-ld.org/playground/" target="_blank">https://edge.m-ld.org/playground/#domain=</a>
<input type="text" size="44" oninput="document.getElementById('playground-link').setAttribute('href', `https://edge.m-ld.org/playground/#domain=${this.value}`)"/>

Now, when you input a Todo in the app and hit \<Enter\>, you'll see data appearing in the playground:

![m-ld playground showing a single Todo](img/playground-success.png "m-ld playground showing a single Todo")

Note that the data doesn't exist anywhere other than in your demo app, and now in the playground. By using the playground, you've created your first _clone_ of the data. It just happens that the playground is actually a completely different app. This is a key feature of Sustainable Web Apps, that the data graphs can be shared to other apps, live.


<a href="https://flems.io/#0=N4IgZglgNgpgziAXAbVAOwIYFsZJAOgAsAXLKEAGhAGMB7NYmBvAHgEIARAeQGEAVAJoAFAKIACEmQB8AHTQtJUMVAxoA5gF4ZIJttloZxBTAwATfYcMscxDGOqEMAJzgxiWkAFdiYALQAObTEAegtiKxs7TBwPADcIGAB3AAdaJ3CQe3pGBg9EiFNiQg1TGHjqGF98wsIKMQg0CGIIDChfOGpWmA0ARiDQuUsjZuJYKT4YLGSVRjFAIgIxPlpTWgBZADUeFmCRscGjYMITc32WACNlgE8wq1dqZvp7FTg4D2Jl2gxk5KCCjy+fiAbuEjEczDAnE8MC8PGDSk49PshlZCD0pCw7IQnDAwB58Hp3is4NsMOjDmikSCrA1kt5KcjiNMMBVCLQoPCPAB1RzEMRoGAwUxwMTvMRnGBiFb8gD82npVNs3loYFo1E8cHlQ2oz1e2n5iV8hNocoMCqGfz1SRNDKGpggcAwZ1gpk1xAGpqG2zhEP0Yj9Ym2dweaDC2wupmuci9pCg+koIFcsHuEHoCEQIAATD1EAAGEAAXwo6GwuHT+AAVggqHQGExiHgIFM0rzgNr6DA6p5PAV82IwE5aFgxAByEiMuCIYLBSv4LC+dn4NJqYIwAAeboapVXs8rw4A3HJG6l0mJgKtJmlLgAZMowKC9-uDkdj5ITqczucLpcr9fBHBYS951vKAdzgfdDybE9gAASVoAAlC9GDgB8ByHUdiHHSdpzgWd51MRcnGXNc3TgVUAGs3HwFNQPAgw0BrOBeRrSA1DEDRTzkP1tAAAQKbREDELsCgACgASgoTixB4lYsAwBp+LEAADAASYAhNMMT83wWknQgah8DURJcK-QjFIktA-TUJh4HtATiCcTwO0klMBLUpwIAExSXzfYJDOM-Cl0U-M5HzA96NTXkANKJR2IwRI5KYqB22E-UxHPACnGvYC6lghCAKQupmIgNRRLChi2RgfAkrUYTFNWZY73sbEMEYUwxBasRVKKtRkGHbiZLktBhwAXXzRTSsGcK0EYvkkmgtBaV5diVjVHAGAMtwRFgNbiAAIUuaCNMtRJtAmtB9Xmxb8DtB0nUFNi+1aVwwrkC6Fu8fAzFMERYjrK97RyCFhO0CjLk8QE6mEmBRLYqQOIs+owDEKH8FBtiNHY7QRFrBFMgAMjx2bEkuj7YlaRyqqYNQihh4BJL9KK73wRJ3MYYS6YR-0RSaWABLeq6yagRzzK5-NRPwTpiAcYTytgfAIQHJxSq5rn6ZCYJFiOCztRMSEiglGlvDFWhVz7NIRSOWb13qRgsDV-nSfJiVMZAbQwr9YK0DFvd40TGBk1TPAemzHNfGDxAAFYAGYC2GqgoAaMi01QEBolLEBNzXCWXnjTwnHIdNvKwpw4oMppCE8M51QhGscmICXBz-PCm-ZQ0PiwWJqF8MnGigFRK2CbEAN+9p3mxYJqBeYJM+3SeqxAYhLmSdPGMuWBY-zIA" target="_blank"><img src="flems.svg" height="30"> <b>New Todo handler, on Flems</b></a>