# Introduction
FoundryVTT inherently runs on javascript. Javascript is a fairly common language for web development for which many tutorials exist online already and covering these would be outside the scope of this guide. As such it will not go into details on generic javascript concepts like objects, arrays and functions and instead focus on things specific to FoundryVTT’s architecture and how to use macros to manipulate things inside your game. Where relevant links to external websites might be included to provide explanation on what javascript inherent, but not Foundry specific, functions like for example `filter()` do. 

Since Foundry can not create worlds without using a specific system some things you will see in screenshots may be specific to the dnd5e system. The general principles remain the same though regardless of system so extrapolation to your system of choice should be possible.
Furthermore this guide is not the only resource that exists. For example you can find many commented code examples here.
And this message on the FoundryVTT Discord goes in more detail regarding asynchronous functions.
A short overview of Foundry’s Application class can be seen [here](https://gitlab.com/Freeze020/foundry-vtt-scripts/-/tree/master/api-examples).

**IMPORTANT**: If you wish to follow along, create a new world to play around in. You will eventually break things irreparably and when you do so it is important that you can just delete the broken world without any issue.

# The console
Especially when it comes to inspecting data structures, what your code is actually calculating, where a function is designed and similar things the console is a useful and near mandatory tool. As such we will go over how to open and actually the console before diving into more complicated topics.

The first step to actually using the console is to open it. To do this press F12 (Command-option-i if you are on Mac, if that doesn’t work fn+F12 might), this opens the developer’s tools. Now navigate to the tab called console, marked by a red arrow in the image below.

***INSERT IMAGE***

Whenever you wish to enter anything into the console you will want to use the text box at the bottom:

***INSERT IMAGE***

The console is of vital importance anytime you want to interact with Foundry’s data structure. In this guide we will mostly make use of its autocomplete functionality and use it to “dig” through existing data structures.
The autocompletion portion is fairly self explanatory. Anytime you enter anything you will start getting suggestions on what you might end up typing. For example if we just type `_tok` we get the suggestion of `_token`.

***INSERT IMAGE***

Taking the suggestion of `_token` and hitting enter we get something like what is shown in the next screenshot. This is the last (not current!) selected token placeable. As such it might be `null` if you have not selected a token since launching the world. In that case please select a token first before hitting enter.

***INSERT IMAGE***

While we can already see a couple fields of the token we will have to expand what the console returned to inspect it further. For that click the little white arrow on the left side.

***INSERT IMAGE***

After expanding we are greeted with much more information that we can expand further should the need arise but for now this suffices as a short introduction into using the console. You can do much more interesting things like looking at what arguments functions take, directly jumping to the line of code they are declared in, etc. but most of that is outside the scope of this guide.

***INSERT IMAGE***

# Basic foundry infrastructure
**IMPORTANT:** In case you missed the warning at the start it is highly recommended to make a new world to play around in should you wish to follow along as you will eventually break things irreperably!
## Updating sidebar documents
Now that we went over how to use the console let's start with some basic changes to existing documents (read on) and the underlying infrastructure we utilize for this.

Nearly everything you will want to modify in Foundry is a document. That includes chat messages, combats, combatants, actors, items, journal entries, cards, decks, playlists, sounds et cetera. Many of these stand in various relations to each other which I will go over in the second section of this chapter but for now let's take a look at how you would make lasting changes to an actor hereafter referred to as an update (since that’s the name of the relevant function).

The first step of every change to a document is to actually get the document we want to modify.

Nearly everything you will ever need can be referred to from the `game` object. If you just want to look around in there you can just look at it like we did in the previous chapter on console usage. As such entering game into the console gives us something like this:

***INSERT IMAGE***

As you can see we have a property `actors` in there which contains every actor in the sidebar. While digging around in there is all well and good you usually only want to modify a specific actor. As such we need some way to actually grab only that specific actor. 
For this you usually use either the `get()` or `getName()` functions which get you the document based on the passed id or name respectively. In the case of actors we also have a third method which utilizes the built in macro helper variables. When you execute a macro and have a token selected you can refer to the selected token via the variable `token` and the associated actor either directly via `actor` or for unlinked tokens `token.actor`. Note that this does not work in the console, only a macro. For more information on the relation between tokens and actors see the dedicated section later in this chapter but for now let's keep things general and get an actor from the sidebar (again referred to via `game.actors`) and use the `getName()` function to make things easy to parse.
Start by making a new macro (ensure the type is set to script!) and then entering the below code. For our example I created an actor in the sidebar called Steve. So to refer to that actor in our macro we would do:
```js
game.actors.getName("Steve")
```

If we want to take a closer look at Steve in the console we would pass it to `console.log()`. To make things easier later we will also assign him to a variable which I arbitrarily decided to call `wantedActor`. As such our code now looks like this:
```js
const wantedActor=game.actors.getName("Steve") // This assigns steve to a variable
console.log(wantedActor) // This throws Steve into the console so we can look at him closer
```

Since the structure of actors and items is largely dependent on the system you are using your output might look slightly different but you should get something similar to this:

***INSERT IMAGE***

Now we have a whole bunch of fields but let's go over some interesting ones. In most cases you will want to update system specific data such as the hp of an actor. From v10 onward this system specific data is found in the `system` field which varies from system to system and as such won't be discussed in detail here. 
However modifying a document is pretty much the same no matter what field you want to modify. In all cases you call the `update()` method on your document and pass it an object with key:value pairs where the key is a string referring to the "path" to the property and the value is the value you wish to assign to the relevant property. Since systems often add a bunch of “shortcuts” to the actor which are derived from, for example the items the actor has, we will want to look at just the data of the actor itself. To do so we modify our previous code to reduce the complex actor document to just an object and throw it in the console once again:
```js
const wantedActor=game.actors.getName("Steve") // This assigns steve to a variable
console.log(wantedActor.toObject()) // This throws Steve's data into the console so we can look at him closer
```
This should give us something like this:

***INSERT IMAGE***

If you compare it to what we had previously you will notice that it’s alot cleaner and that all the helpful functions are missing. For our purposes of just figuring out what data the actor actually has this is perfect though.
As a practical example, let's assume we know the name of an actor in the sidebar (in this case it’s called Steve) and we want to change the image of said actor. So first we again get the actor, reduce it to an object, log it and dig through the data to find what we need. Through trial and error or logical deduction we realize that the image the actor is using is saved in the img attribute of the actor.

***INSERT IMAGE***

Since the `img` property is directly located in the root of the actor’s data the path for our update object would be just img. If we now want to set the image to the following example path of `"icons/svg/dice-target.svg"` our updates object containing the change we wish to apply would look like this:
```js
const change={"img":"icons/svg/dice-target.svg"}
```

For ease of use I assigned it to a variable called change but you can obviously skip that part if you want.
Now to actually change the actor it via name from the sidebar, construct our update object and then actually call the update (which we await since `update()` like any database operation is asynchronous).

```js
const wantedActor=game.actors.getName("Steve") // Get our actor from the ones in the sidebar based on the name
const change={"img":"icons/svg/dice-target.svg"} // Construct the update object containing our changes
await wantedActor.update(change) // Call the update and wait for it to finish
```
Once you execute this you will notice that Steve now has a nice new image!

Often you want to change system specific data though which as stated previously is located within the `system` property of the actor. The process is nearly identical however. We again get the actor, log the data, dig through it to find what we want, construct our update object and then call  the update.
So for example if we were to dig in a dnd5e actor searching for how to change the current hp value of the actor to 1 we would end up with something like this for our update object:
```js
let change = {"system.attributes.hp.value":1}
```

The only difference from our image example is that we now have a path that goes deeper than just one property which is why we just chain them all together connecting the different steps with dots. For reference this is what the data looks like in the console:

***INSERT IMAGE***

Now the values for our update object don't have to be plain values they can be variables like anything else so if we would want to set the hp of the actor to be one point lower than it is before calling the macro we would do

```js
let wantedActor=game.actors.getName("xyz") // get the actor from the ones in the sidebar
let newValue=wantedActor.system.attributes.hp.value-1 //calculate the new hp value
let change={"system.attributes.hp.value":newValue} // construct our update object with the calculated hp value
await wantedActor.update(change) // actually call the update
```

What we are doing here is to again get the actor first, calculate the new hp value based on the current hp value, construct our update object and then actually update the actor applying the change.

With this structure you can change any document in the sidebar. Case in point, let's change the image of an item in the sidebar named `"Dagger"` from its current image to `"icons/svg/blood.svg"`. First of all we need to figure out a way to get all items in the sidebar. Digging through the game object reveals that they are contained in the `items` property of said object (so `game.items`).

***INSERT IMAGE***

We then again get the document by name, dig around it to see what the image is saved under (omitted here but you'd use `console.log()` like before), realize the image is again saved in the `img` property, construct our change and finally update the item. In code terms this would look like this.

```js
let wantedItem=game.items.getName("Dagger") // Get the item from all items in the sidebar by name
let change = {"img":"icons/svg/blood.svg"} // Construct our update object
await wantedItem.update(change) // call the update to actually change stuff
```

## Embedded Documents
Now you might say that's cool and all but I want to update an item on the actor not in the sidebar!
This brings us to our next topic: embedded documents. As mentioned at the start nearly everything you will modify is a document and moving an item from the sidebar onto an actor creates a new item embedded in the actor. So now we have documents(items) inside a document(actor) ergo an embedded document. Since the item we want isn't in the sidebar we won't find it in `game.items` but we will find it in the `items` property of our actor.
So to make changes to a specific item on a specific actor we would first need to get the actor then get the item on the actor then construct our update object and lastly call the update like before. So effectively we are just going one layer deeper than before which in practice looks like this, though this time the Dagger is embedded in Steve and not in the sidebar!

```js
let wantedActor=game.actors.getName("Steve") // Get the actor from our sidebar actors
let wantedItem=wantedActor.items.getName("Dagger") // get the item from the items on our actor
let change={"img":"icons/svg/blood.svg"} // construct our update object
await wantedItem.update(change) // update the item
```

### Filter and find
The other document embedded on an actor you might encounter on an actor is an active effect which in v10 has a quirk in the dnd5e system in that the name property of the document is not actually used meaning that `getName()` is not a valid way to get it. This brings us to the next point on the list which is additional ways to get specific documents out of a collection of documents. Where `get()` and `getName()` isn't applicable you can use `find()` and `filter()` to find the first document that matches the condition or filter out all documents that match the condition respectively.

Since these functions are not Foundry specific (and indeed mirror the ones for basic Arrays in javascript) only a few Foundry specific examples will be presented here while the following links should be referred to for explanation on the [find](https://developer.mozilla.org/de/docs/Web/JavaScript/Reference/Global_Objects/Array/find) and [filter](https://developer.mozilla.org/de/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) functions.

For the aforementioned active effects an often used property to find a specific effect would be `label` which contains an easily readable string.
So to find a specific effect called `"Blinded"` on a specific actor called `"Steve"` who is located in the sidebar we would write:
```js
let wantedActor=game.actors.getName("Steve") // Get actor from sidebar by name
let wantedEffect=wantedActor.effects.find(i=>i.label==="Blinded") // Get the effect on the actor by name
```

From there we can log, update, etc the effect like before since it too is just a document.

A good example for filter would be to get all items on a specific actor by type. Type contains whether the item is a spell, feature or otherwise in the dnd5e, pf2e and other systems and as such is something you often filter by. While various systems have various item types the overall structure would still be the same
So to filter out the items with a type of spell on a specific actor called `"Steve"` you would do:

```js
let wantedActor=game.actors.getName("Steve") // Get actor from sidebar by name
let wantedEffect=wantedActor.items.filter(i=>i.type===spell") // Filter the items on an actor based on having a type of "spell"
```

With this you should be able to get and modify documents in the sidebar or embedded in other documents and apply changes to each of them. While you can do this individually for each item, often it is advantageous to do all the updates at once which is something the later chapter on “batch changes” will go over.
### Documents embedded in embedded documents
As a final note on embedded documents before we move over to the next section:
You might have noticed that you can embed effects on items and you can embed items on an actor. As such you can have an effect embedded on an item embedded on an actor which is effectively a document embedded in an embedded document. Don't worry it's not only confusing for you but also for Foundry. As such you can NOT edit these double embedded documents barring rather complicated methods that would go over the scope of this guide.
For now just work around this limitation by adding the effects on items in the sidebar or before they are even created on the actor (more on creating documents in the dedicated section).
This limitation will be remedied in v11 however at which point this small section of the guide will be removed.
## Documents vs. Placeables
You might have noticed that so far documents embedded in the scene (namely tokens, walls, lights, sounds, map notes, drawings and templates) have not been discussed. That is because dealing with those documents requires knowledge about the difference between placeables and documents. The former are the things you actually interact with on the canvas while the latter actually hold the data and can be easily modified as we established previously. 

Now why even bother with the placeables then? Afterall you want the document in the end anyhow.

The reason we need to touch placeables at all is that most functions to easily get specific scene elements are functions of the canvas which holds placeables not documents. The reason for needing those functions is that most documents embedded in a scene don’t have a name (tokens are the exception), so `getName()` falls flat, and `get()` requires an id. While you can get the id by left clicking the little book icon in the token’s sheet header (see picture below), oftentimes you don’t want this specific token you got the id of a while back, but rather whatever token you currently have selected.

***INSERT IMAGE***

Now with that out of the way let’s take a look at the previously alluded to functions.
For walls, tokens,drawings and tiles `canvas.placeholderThatDependsOnTheDocument.controlled` gives you an array containing the placeables of the selected specified document.
So `canvas.walls.controlled` would be an array holding all selected wall placeables, `canvas.tokens.controlled` would be an array holding all selected token placeables, `canvas.drawings.controlled` would be the selected drawings and `canvas.tiles.controlled` an array of all selected tiles.
As you might have noticed templates, lights, ambient sounds, map notes were not in the list above. The reason for that is that you can’t control/select them, bringing us to the next function or helper attribute: `hover` which returns you the currently hovered placeable. Since your mouse is busy when you are hovering over something you’d execute this via hotkey from your macro hotbar. As before, which kind of placeable you are getting can be inferred from the command itself.

`canvas.templates.hover` is the currently hovered template, `canvas.lighting.hover` is the currently hovered light source, `canvas.sounds.hover` is the currently hovered ambient sound and `canvas.notes.hover` is the currently hovered note.
Getting the document from a placeable is also very simple as each placeable has a document attribute which links to the relevant document. As such canvas.lighting.hover.document would be the document of the currently hovered light source placeable which you can then update as discussed previously for actors and items. As always it is advisable to log the document in the console first to see the different fields it has.
## Batch changes
Sometimes you want to update a bunch of documents at once and while a loop archives this it is both slow and non-optimal in many ways. As such Foundry has a large number of functions to update many things at once. The ones you’ll most often use are `updateDocuments()` which updates multiple documents that (unless you specify it) are not embedded in another document. For embedded documents you’d usually use `updateEmbeddedDocuments()`, which updates multiple embedded documents. It is relevant to note, that you can also use both functions to update a single document.

Let’s start with the syntax for updateDocuments.
`updateDocuments()` has to be called on the relevant class extending ClientDocumentMixin which is a fancy way of saying “call it on what type of document you want to update”. 
`Actor.updateDocuments()` would update multiple actors for example while `Item.updateDocuments()` would update multiple items. Use the console to figure out the name for the other classes, they are relatively easy to find. The function itself accepts an array of update objects which are structured nearly identically to the one passed to the `update()` function (which updates a single document) with one exception: the `_id` field. This field holds the id of the document that is to be updated. So let’s take the following example:
Let’s say we have two actors in our sidebar we want to update to the same new image at once `"Steve"` and `"Tom"`. We get Steve and Tom by name, construct our update objects and then call `Actor.updateDocuments()` which accepts an array of update objects. This would look as follows:
```js
let steve=game.actors.getName("Steve") // get the document for steve
let tom=game.actors.getName("Tom") //get the document for tom
arrayOfUpdateObjects=[ // make our array of updates
   {
      _id:steve.id,//specify the id of the document we want to update
      img:"Cool/Path/here/xyz" // specify the path we want to set img to
   },{ // do the same for tom
      _id:tom.id,
      img:"Cool/Path/here/xyz"
   }
]
Actor.updateDocuments(arrayOfUpdateObjects) // Actually update them
```

While it isn’t really needed to call `updateDocuments()` for an update to only two actors the improved performance becomes noticeable if you have a lot of actors. For example, let's say that we would like to update all actors with the “npc” type in the sidebar to the same image. In that case we would do something like this:
```js
let relevantActors=game.actors.filter(i=>i.type==="npc")
let updates=relevantActors.map(i=>({
_id:i.id,
img:"Cool/Image/xyz"
}))
Actor.updateDocuments(updates)
```
If you can’t remember how [map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) and [filter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) worked check out the relevant documentation on them online.

Now `updateEmbeddedDocuments()` is very similar though a few differences exist. For starters you don’t call the function on the document class itself but rather the document the documents you want to update are embedded on. Furthermore since you may have multiple different types of embedded documents (items as well as effects in the case of actors) you need to specify what type of embedded document your update array is meant for.

Let’s do this for tokens embedded in the currently viewed scene. Assuming that the canvas is not disabled you can access the currently viewed scene via `canvas.scene`. The tokens that are embedded in the scene can be found in scene.tokens so `canvas.scene.tokens` gives us all the token documents of the currently viewed scene. As an example we will update the image of all tokens that currently have the default mystery man image to a different image. So we need to:
- Filter out all tokens that have the mystery man icon
- Make an array of update objects
- Then pass that to `updateEmbeddedDocuments().`

To this end we actually need to know where the image of a token is saved in its data. You’d get this by digging in the data like before. You’ll eventually find that the token’s image is under `texture.src`.

Therefore the code would look like this
```js
const allTokens=canvas.scene.tokens // All the tokens o
n the scene
const mysteryTokens=allTokens.filter(i=>i.texture.src==="icons/svg/mystery-man.svg") // filter out all tokens that have the mystery man icon
const updates=mysteryToken.map(i=>({ // construct the update array
_id:i.id,
"texture.src":"icons/svg/bones.svg"
}))
canvas.scene.updateEmbeddedDocuments("Token",updates)// call updateEmbeddedDocuments where the first argument is the type of document and the second is the update array
```

## Creating documents
Aside from modifying existing documents via `update()` and its siblings, you might also want to create brand new documents.
In order to create documents you want to call the `create()` function to create a single document or `createDocuments()` in case you want to batch the creation of multiple. Similar to before you just pass an object of data to `create()` or an array of such objects to `createDocuments()`. Instead of using that data to modify an existing document, it is then used to create a new one. Note that in both cases you want to call the method on the document’s class itself so:
```js
Item.createDocuments(
[{
name:"Dagger",
type:"weapon"
}]
)
```
Would create a new item of the `"weapon"` type with the name `"Dagger"` in the sidebar. As before types are system specific so look at an existing document that you want to recreate to know what type you should specify. Similarly creating embedded documents can be done with `createEmbeddedDocuments()` though the first argument would again be the document type you want to create.
As such the following would first create an actor with the name of `"Steve"` and a type of `"npc"`, and then add an item with the type of `"weapon"` and a name of `"Dagger"` to it.
```js
const steve=await Actor.create({ // Awaiting the creation is nessecary so that the actor exists by the time we try to create the items
type:"npc",
name:"Steve"
})

await steve.createEmbeddedDocuments("Item",[{
name:"Dagger",
type:"weapon"
}]
)
```
## Deleting Documents
When you can create documents you will also want to delete them. This is done via the `delete()`, `deleteDocuments()` and for embedded Documents: `deleteEmbeddedDocuments()` methods.
In the case of `delete()` you’d just call it on the document itself. As such:
```js
game.actors.getName("Steve").delete()
```

would delete the actor in the sidebar named "Steve". If you want multiple you will have to pass an array of ids to `deleteDocuments()`. So if we want to delete all actors in the sidebar that are not in a folder we would need to:
- Filter out the actors in the sidebar that don’t have a folder
- Save the ids of those actors to an array
- Pass that array to `deleteDocuments()`
This would look as follows:
```js
const noFolderItems=game.items.filter(i=>!i.folder) // filter out the items that aren't in a folder
const deleteIds=noFolderItems.map(i=>i.id) // map the array of items to an array of ids
Item.deleteDocuments(deleteIds) // actually delete the items.
```

Analogously for items embedded on our example actor steve we would filter out all the items we want to delete, let’s say all weapons, map those to ids, and then pass those ids to the function:
```js
const steve=game.actors.getName("Steve") // get our actor
const weapons=steve.items.filter(i=>i.type==="weapon") // filter out all weapons
const deleteIds=weapons.map(i=>i.id) // get an array of item ids to delete
steve.deleteEmbeddedDocuments("Item",deleteIds) // delete all items with the passed ids on steve
```
## Tokens, actors and synthetic actors
On a final slightly more advanced note let’s talk about tokens and their relationship with actors.
As you may know each token on the scene is used to represent an actor. If you remove the associated actor from the sidebar the token breaks and double clicking it throws an error telling you that it can no longer find its token. Similar to how we can double click a token to open the associated actor’s sheet, we can refer to the actor associated with a token in a macro by accessing the actor property of the token. So `token.actor` would give us the actor document associated with the token. Now some of you may have noticed that unlinked tokens exist, for which the actor associated with the token differs from the “source” actor in the sidebar. This is what we call a synthetic actor which has a bunch of quirks. Which we will go over shortly. Most of these are purely of academic interest though so should this not be of interest to you, just take away that you want to use token.actor to get the actor of a relevant token.

The actors associated with unlinked tokens is what we call a “synthetic actor” because it is computed from the source actor in the sidebar and data on the token telling foundry what should be different from the sidebar actor. You can find this data in the `actorData` property of the token though unless you know what you are doing you probably shouldn’t mess with it, instead updating the actor you get from `token.actor` like usual. The reason this might be of importance to you however is that the id of that synthetic actor is identical to the one in the sidebar, however unlike the source actor it isn’t in the sidebar so you can’t go via `Actor.updateDocuments()` as the id you would pass it would just target the sidebar actor. This leads to many edge cases and confusing situations, so if you ever notice that you are changing the sidebar actor when you want a token’s synthetic actor, remind yourself that you want to go via `token.actor`.
