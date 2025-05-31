# Introduction
FoundryVTT inherently runs on javascript. Javascript is a fairly common language for web development for which many tutorials exist online already and covering these would be outside the scope of this guide. As such it will not go into details on generic javascript concepts like objects, arrays and functions and instead focus on things specific to FoundryVTT’s architecture and how to use macros to manipulate things inside your game. Where relevant links to external websites might be included to provide explanation on what javascript inherent, but not Foundry specific, functions like for example `filter()` do. 

Since Foundry can not create worlds without using a specific system some things you will see in screenshots may be specific to the dnd5e system. The general principles remain the same though regardless of system so extrapolation to your system of choice should be possible.
Furthermore this guide is not the only resource that exists. For example you can find many commented code examples [here](https://gitlab.com/Freeze020/foundry-vtt-scripts/-/tree/master/api-examples).
And this [message on the FoundryVTT Discord](https://discord.com/channels/170995199584108546/699750150674972743/1039547867804745768) goes in more detail regarding asynchronous functions.
A short overview of Foundry’s Application class can be seen [here](https://gitlab.com/Freeze020/foundry-vtt-scripts/-/tree/master/api-examples).
For a guide specifically on Regions see [here](https://github.com/GamerFlix/foundryvtt-api-guide/blob/main/region_macros_guide.md)

This guide has been updated for v12. If you find any inaccuracies please open an issue. It should also be up to date for v13 as none of the changes should have an impact on this guide. Again please open an issue should you find something to the contrary.
While there are only minor differences a v11 version can be found [here](https://github.com/GamerFlix/foundryvtt-api-guide/blob/v11/macro_guide.md).

**IMPORTANT**: If you wish to follow along, create a new world to play around in. You will eventually break things irreparably and when you do so it is important that you can just delete the broken world without any issue.

# The console
Especially when it comes to inspecting data structures, what your code is actually calculating, where a function is designed and similar things the console is a useful and near mandatory tool. As such we will go over how to open and actually the console before diving into more complicated topics.

The first step to actually using the console is to open it. To do this press F12 (Command-option-i if you are on Mac, if that doesn’t work fn+F12 might), this opens the developer’s tools. Now navigate to the tab called console, marked by a red arrow in the image below.

![image7](https://github.com/GamerFlix/foundryvtt-api-guide/assets/62909799/a378d9aa-0c80-4af8-ab93-5dc8c0470e7b)

Whenever you wish to enter anything into the console you will want to use the text box at the bottom:

![image11](https://github.com/GamerFlix/foundryvtt-api-guide/assets/62909799/6429ad25-dd49-427a-a854-e801a5d46822)

The console is of vital importance anytime you want to interact with Foundry’s data structure. In this guide we will mostly make use of its autocomplete functionality and use it to “dig” through existing data structures.
The autocompletion portion is fairly self explanatory. Anytime you enter anything you will start getting suggestions on what you might end up typing. For example if we just type `_tok` we get the suggestion of `_token`.

![image12](https://github.com/GamerFlix/foundryvtt-api-guide/assets/62909799/13e6402a-136d-4eb8-a75b-0d4c2106d0c5)

Taking the suggestion of `_token` and hitting enter we get something like what is shown in the next screenshot. This is the last (not current!) selected Token Placeable. As such it might be `null` if you have not selected a Token since launching the world. In that case please select a Token first before hitting enter.

![image3](https://github.com/GamerFlix/foundryvtt-api-guide/assets/62909799/692ad4d6-a8b1-4326-9a47-f5c7c28fd3c9)

While we can already see a couple fields of the Token we will have to expand what the console returned to inspect it further. For that click the little white arrow on the left side.

![image9](https://github.com/GamerFlix/foundryvtt-api-guide/assets/62909799/7f0fb741-9b54-41f1-95b9-4d49f59997a3)

After expanding we are greeted with much more information that we can expand further should the need arise but for now this suffices as a short introduction into using the console. You can do much more interesting things like looking at what arguments functions take, directly jumping to the line of code they are declared in, etc. but most of that is outside the scope of this guide.

![image10](https://github.com/GamerFlix/foundryvtt-api-guide/assets/62909799/5a4af180-c156-452b-a355-edd728b4a728)

# Basic foundry infrastructure
**IMPORTANT:** In case you missed the warning at the start it is highly recommended to make a new world to play around in should you wish to follow along as you will eventually break things irreperably!
## Updating sidebar Documents
Now that we went over how to use the console let's start with some basic changes to existing Documents (read on) and the underlying infrastructure we utilize for this.

Nearly everything you will want to modify in Foundry is a Document. That includes Chat Messages, Combats, Combatants, Actors, Items, Journal Entries, Cards, Decks, Playlists, Ambient Sounds et cetera. Many of these stand in various relations to each other which I will go over in the second section of this chapter but for now let's take a look at how you would make lasting changes to an Actor hereafter referred to as an update (since that’s the name of the relevant function).

The first step of every change to a Document is to actually get the Document we want to modify.

Nearly everything you will ever need can be referred to from the `game` Object. If you just want to look around in there you can just look at it like we did in the previous chapter on console usage. As such entering game into the console gives us something like this:

![image8](https://github.com/GamerFlix/foundryvtt-api-guide/assets/62909799/1a96a2ad-4908-47b8-8bae-ec179802019d)

As you can see we have a property `actors` in there which contains every Actor in the sidebar. While digging around in there is all well and good you usually only want to modify a specific Actor. As such we need some way to actually grab only that specific Actor. 
For this you usually use either the `get()` or `getName()` functions which get you the Document based on the passed id or name respectively. In the case of Actors we also have a third method which utilizes the built in macro helper variables. When you execute a macro and have a Token selected you can refer to the selected Token via the variable `token` and the associated Actor either directly via `actor` or for unlinked Tokens `token.actor`. Note that this does not work in the console, only a macro. For more information on the relation between Tokens and Actors see the dedicated section later in this chapter but for now let's keep things general and get an Actor from the sidebar (again referred to via `game.actors`) and use the `getName()` function to make things easy to parse.
Start by making a new macro (ensure the type is set to script!) and then entering the below code. For our example I created an Actor in the sidebar called Steve. So to refer to that Actor in our macro we would do:
```js
game.actors.getName("Steve")
```

If we want to take a closer look at Steve in the console we would pass it to `console.log()`. To make things easier later we will also assign him to a variable which I arbitrarily decided to call `wantedActor`. As such our code now looks like this:
```js
const wantedActor=game.actors.getName("Steve") // This assigns steve to a variable
console.log(wantedActor) // This throws Steve into the console so we can look at him closer
```

Since the structure of Actors and Items is largely dependent on the system you are using your output might look slightly different but you should get something similar to this:

![image13](https://github.com/GamerFlix/foundryvtt-api-guide/assets/62909799/9600023b-04fd-426e-aee6-eca146f95bef)

Now we have a whole bunch of fields but let's go over some interesting ones. In most cases you will want to update system specific data such as the hp of an Actor. From v10 onward this system specific data is found in the `system` field which varies from system to system and as such won't be discussed in detail here. 
However modifying a Document is pretty much the same no matter what field you want to modify. In all cases you call the `update()` method on your Document and pass it an Object with key:value pairs where the key is a string referring to the "path" to the property and the value is the value you wish to assign to the relevant property. Since systems often add a bunch of “shortcuts” to the Actor which are derived from, for example the items the Actor has, we will want to look at just the data of the Actor itself. To do so we modify our previous code to call the Document specific function `toObject` and then log the return value instead of the whole Document. This is useful as `toObject()` reduces the complex Actor Document to just a simple Object. The resulting code looks like this:
```js
const wantedActor=game.actors.getName("Steve") // This assigns steve to a variable
console.log(wantedActor.toObject()) // This throws Steve's data into the console so we can look at him closer
```
This should give us something like this:

![image5](https://github.com/GamerFlix/foundryvtt-api-guide/assets/62909799/e4d58496-1b51-49c1-aca8-79409c02a7ef)

If you compare it to what we had previously you will notice that it’s alot cleaner and that all the helpful functions are missing. For our purposes of just figuring out what data the Actor actually has this is perfect though.
As a practical example, let's assume we know the name of an Actor in the sidebar (in this case it’s called Steve) and we want to change the image of said Actor. So first we again get the Actor, reduce it to an Object, log it and dig through the data to find what we need. Through trial and error or logical deduction we realize that the image the Actor is using is saved in the img attribute of the Actor.

![image1](https://github.com/GamerFlix/foundryvtt-api-guide/assets/62909799/64928315-84d5-4623-a9a8-b368605a1fef)

Since the `img` property is directly located in the root of the Actor’s data the path for our update Object would be just img. If we now want to set the image to the following example path of `"icons/svg/dice-target.svg"` our updates Object containing the change we wish to apply would look like this:
```js
const change={"img":"icons/svg/dice-target.svg"}
```

For ease of use I assigned it to a variable called change but you can obviously skip that part if you want.
Now to actually change the Actor it via name from the sidebar, construct our update Object and then actually call the update (which we await since `update()` like any database operation is asynchronous).

```js
const wantedActor=game.actors.getName("Steve") // Get our actor from the ones in the sidebar based on the name
const change={"img":"icons/svg/dice-target.svg"} // Construct the update object containing our changes
await wantedActor.update(change) // Call the update and wait for it to finish
```
Once you execute this you will notice that Steve now has a nice new image!

Often you want to change system specific data though which as stated previously is located within the `system` property of the Actor. The process is nearly identical however. We again get the Actor, log the data, dig through it to find what we want, construct our update Object and then call  the update.
So for example if we were to dig in a dnd5e Actor searching for how to change the current hp value of the Actor to 1 we would end up with something like this for our update Object:
```js
let change = {"system.attributes.hp.value":1}
```

The only difference from our image example is that we now have a path that goes deeper than just one property which is why we just chain them all together connecting the different steps with dots. For reference this is what the data looks like in the console:

![image2](https://github.com/GamerFlix/foundryvtt-api-guide/assets/62909799/1cd7a516-78f6-41b5-9f8e-3f4e6b8b7b1a)


Now the values for our update Object don't have to be plain values they can be variables like anything else so if we would want to set the hp of the Actor to be one point lower than it is before calling the macro we would do

```js
let wantedActor=game.actors.getName("xyz") // get the actor from the ones in the sidebar
let newValue=wantedActor.system.attributes.hp.value-1 //calculate the new hp value
let change={"system.attributes.hp.value":newValue} // construct our update object with the calculated hp value
await wantedActor.update(change) // actually call the update
```

What we are doing here is to again get the Actor first, calculate the new hp value based on the current hp value, construct our update Object and then actually update the Actor applying the change.

With this structure you can change any Document in the sidebar. Case in point, let's change the image of an Item in the sidebar named `"Dagger"` from its current image to `"icons/svg/blood.svg"`. First of all we need to figure out a way to get all Items in the sidebar. Digging through the game Object reveals that they are contained in the `items` property of said Object (so `game.items`).

![image4](https://github.com/GamerFlix/foundryvtt-api-guide/assets/62909799/ab71a9e8-bac3-45c9-bc03-91124509f04a)

We then again get the Document by name, dig around it to see what the image is saved under (omitted here but you'd use `console.log()` like before), realize the image is again saved in the `img` property, construct our change and finally update the Item. In code terms this would look like this.

```js
let wantedItem=game.items.getName("Dagger") // Get the item from all items in the sidebar by name
let change = {"img":"icons/svg/blood.svg"} // Construct our update object
await wantedItem.update(change) // call the update to actually change stuff
```

## Embedded Documents
Now you might say that's cool and all but I want to update an Item on the Actor not in the sidebar!
This brings us to our next topic: embedded Documents. As mentioned at the start nearly everything you will modify is a Document and moving an Item from the sidebar onto an Actor creates a new Item embedded in the Actor. So now we have Documents(Items) inside a Document(Actor) ergo an Embedded Document. Since the Item we want isn't in the sidebar we won't find it in `game.items` but we will find it in the `items` property of our actor.
So to make changes to a specific Item on a specific Actor we would first need to get the Actor then get the Item on the Actor then construct our update Object and lastly call the update like before. So effectively we are just going one layer deeper than before which in practice looks like this, though this time the Dagger is embedded in Steve and not in the sidebar!

```js
let wantedActor=game.actors.getName("Steve") // Get the actor from our sidebar actors
let wantedItem=wantedActor.items.getName("Dagger") // get the item from the items on our actor
let change={"img":"icons/svg/blood.svg"} // construct our update object
await wantedItem.update(change) // update the item
```

### Filter and find
The other Document embedded on an Actor you might encounter on an Actor is an ActiveEffect in case you want to delete all currently disabled ActiveEffects on an actor it becomes clear that filtering by name is not a valid option. This brings us to the next point on the list which is additional ways to get specific Documents out of a collection of Documents. Where `get()` and `getName()` isn't applicable you can use `find()` and `filter()` to find the first Document that matches the condition or filter out all Documents that match the condition respectively.

Since these functions are not Foundry specific (and indeed mirror the ones for basic Arrays in javascript) only a few Foundry specific examples will be presented here while the following links should be referred to for explanation on the [find](https://developer.mozilla.org/de/docs/Web/JavaScript/Reference/Global_Objects/Array/find) and [filter](https://developer.mozilla.org/de/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) functions.

For the aforementioned usecase of finding disabled ActiveEffects you could filter them by the `disabled` property which would be `true` in case of a disabled ActiveEffect.
In order to find a single effect which is disabled and embedded in a specific Actor called `"Steve"` who is located in the sidebar we would write:
```js
let wantedActor=game.actors.getName("Steve") // Get actor from sidebar by name
let wantedEffect=wantedActor.effects.find(i=>i.disabled) // Get the effect on the actor by name
```
Since disabled is a boolean itself this would give us the "first" disabled ActiveEffect on the Actor `"Steve"`.
From there we can log, update, etc the effect like before since it too is just a Document.

Since we want to find all disabled ActiveEffects though find doesn't help us much as it only returns a single result. As such we'd use `filter()` to find all Documents matching the condition. So to find all disabled ActiveEffects we'd use:
```js
let wantedActor=game.actors.getName("Steve") // Get actor from sidebar by name
let wantedEffects=wantedActor.effects.filter(i=>i.disabled) // Get the effect on the actor by name
```

Another good example for filter would be to get all Items on a specific Actor by type. Type contains whether the Item is a spell, feature or otherwise in the dnd5e, pf2e and other systems and as such is something you often filter by. While various systems have various Item types the overall structure would still be the same
So to filter out the Items with a type of `"spell"` on a specific Actor called `"Steve"` you would do:

```js
let wantedActor=game.actors.getName("Steve") // Get actor from sidebar by name
let wantedEffect=wantedActor.items.filter(i=>i.type===spell") // Filter the items on an actor based on having a type of "spell"
```

With this you should be able to get and modify Documents in the sidebar or embedded in other Documents and apply changes to each of them. While you can do this individually for each Item, often it is advantageous to do all the updates at once which is something the later chapter on “batch changes” will go over. 
## Documents vs. Placeables
You might have noticed that so far Documents embedded in the scene (namely Tokens, Walls, Ambient Lights, Ambient Sounds, Map Notes, Drawings and Templates) have not been discussed. That is because dealing with those Documents requires knowledge about the difference between Placeables and Documents. The former are the things you actually interact with on the canvas while the latter actually hold the data and can be easily modified as we established previously. 

Now why even bother with the Placeables then? Afterall you want the Document in the end anyhow.

The reason we need to touch Placeables at all is that most functions to easily get specific scene elements are functions of the canvas which holds Placeables not Documents. The reason for needing those functions is that most Documents embedded in a scene don’t have a name (Tokens are the exception), so `getName()` falls flat, and `get()` requires an id. While you can get the id by right clicking the little book icon in the Token’s sheet header (see picture below), oftentimes you don’t want this specific Token you got the id of a while back, but rather whatever Token you currently have selected.

![image6](https://github.com/GamerFlix/foundryvtt-api-guide/assets/62909799/a3bcbf1a-2995-4b58-801b-229bffab8876)

Now with that out of the way let’s take a look at the previously alluded to functions.
For Drawings, Tiles, Tokens, Regions and Walls `canvas.placeholderThatDependsOnTheDocument.controlled` gives you an Array containing the Placeables of the selected specified Document.
So `canvas.drawings.controlled` would be an Array holding all selected Drawing Placeables, `canvas.tiles.controlled` would be an array holding all selected Tile Placeables, `canvas.tokens.controlled` would be the selected Tokens, `canvas.regions.controlled` the selected Regions and `canvas.walls.controlled` an Array of all selected Wall Placeables.
As you might have noticed Templates, Lights, Ambient Sounds and Map Notes were not in the list above. The reason for that is that you can’t control/select them, bringing us to the next helpful getter: `hover` which returns you the currently hovered Placeable. Since your mouse is busy when you are hovering over something you’d execute this via hotkey from your macro hotbar. As before, which kind of Placeable you are getting can be inferred from the command itself.

`canvas.templates.hover` is the currently hovered template, `canvas.lighting.hover` is the currently hovered light source, `canvas.sounds.hover` is the currently hovered ambient sound and `canvas.notes.hover` is the currently hovered note.
Getting the Document from a Placeable is also very simple as each Placeable has a Document attribute which links to the relevant Document. As such `canvas.lighting.hover.document` would be the Document of the currently hovered light source Placeable which you can then update as discussed previously for Actors and Items. As always it is advisable to log the Document in the console first to see the different fields it has.
## Batch changes
Sometimes you want to update a bunch of Documents at once and while a loop archieves this it is both slow and non-optimal in many ways. As such Foundry has a large number of functions to update many things at once. The ones you’ll most often use are `updateDocuments()` which updates multiple Documents that (unless you specify it) are not embedded in another Document. For Embedded Documents you’d usually use `updateEmbeddedDocuments()`, which updates multiple Embedded Documents. It is relevant to note, that you can also use both functions to update a single Document.

Let’s start with the syntax for updateDocuments.
`updateDocuments()` has to be called on the relevant class extending ClientDocumentMixin which is a fancy way of saying “call it on what type of Document you want to update”. 
`Actor.updateDocuments()` would update multiple Actors for example while `Item.updateDocuments()` would update multiple Items. Use the console to figure out the name for the other classes, they are relatively easy to find. The function itself accepts an Array of update Objects which are structured nearly identically to the one passed to the `update()` function (which updates a single Document) with one exception: the `_id` field. This field holds the id of the Document that is to be updated. So let’s take the following example:
Let’s say we have two Actors in our sidebar we want to update to the same new image at once `"Steve"` and `"Tom"`. We get Steve and Tom by name, construct our update Objects and then call `Actor.updateDocuments()` which accepts an Array of update Objects. This would look as follows:
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

While it isn’t really needed to call `updateDocuments()` for an update to only two Actors the improved performance becomes noticeable if you have a lot of Actors. For example, let's say that we would like to update all Actors with the “npc” type in the sidebar to the same image. In that case we would do something like this:
```js
let relevantActors=game.actors.filter(i=>i.type==="npc")
let updates=relevantActors.map(i=>({
_id:i.id,
img:"Cool/Image/xyz"
}))
Actor.updateDocuments(updates)
```
If you can’t remember how [map](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map) and [filter](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array/filter) worked check out the relevant documentation on them online.

Now `updateEmbeddedDocuments()` is very similar though a few differences exist. For starters you don’t call the function on the Document class itself but rather the Document the Documents you want to update are embedded on. Furthermore since you may have multiple different types of embedded Documents (Items as well as effects in the case of Actors) you need to specify what type of embedded Document your update Array is meant for.

Let’s do this for Tokens embedded in the currently viewed scene. Assuming that the canvas is not disabled you can access the currently viewed scene via `canvas.scene`. The Tokens that are embedded in the scene can be found in `scene.tokens` so `canvas.scene.tokens` gives us all the Token Documents of the currently viewed Scene. As an example we will update the image of all Tokens that currently have the default mystery man image to a different image. So we need to:
- Filter out all Tokens that have the mystery man icon
- Make an Array of update Objects
- Then pass that to `updateEmbeddedDocuments().`

To this end we actually need to know where the image of a Token is saved in its data. You’d get this by digging in the data like before. You’ll eventually find that the Token’s image is under `texture.src`.

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

## Creating Documents
Aside from modifying existing Documents via `update()` and its siblings, you might also want to create brand new Documents.
In order to create Documents you want to call the `create()` function to create a single Document or `createDocuments()` in case you want to batch the creation of multiple. Similar to before you just pass an Object of data to `create()` or an Array of such Objects to `createDocuments()`. Instead of using that data to modify an existing Document, it is then used to create a new one. Note that in both cases you want to call the method on the Document’s class itself so:
```js
Item.createDocuments(
[{
name:"Dagger",
type:"weapon"
}]
)
```
Would create a new Item of the `"weapon"` type with the name `"Dagger"` in the sidebar. As before types are system specific so look at an existing Document that you want to recreate to know what type you should specify. Similarly creating embedded Documents can be done with `createEmbeddedDocuments()` though the first argument would again be the Document type you want to create.
As such the following would first create an Actor with the name of `"Steve"` and a type of `"npc"`, and then add an Item with the type of `"weapon"` and a name of `"Dagger"` to it.
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
When you can create Documents you will also want to delete them. This is done via the `delete()`, `deleteDocuments()` and for Embedded Documents: `deleteEmbeddedDocuments()` methods.
In the case of `delete()` you’d just call it on the Document itself. As such:
```js
game.actors.getName("Steve").delete()
```

would delete the Actor in the sidebar named "Steve". If you want multiple you will have to pass an Array of ids to `deleteDocuments()`. So if we want to delete all actors in the sidebar that are not in a folder we would need to:
- Filter out the Actors in the sidebar that don’t have a folder
- Save the ids of those Actors to an Array
- Pass that Array to `deleteDocuments()`
This would look as follows:
```js
const noFolderItems=game.items.filter(i=>!i.folder) // filter out the items that aren't in a folder
const deleteIds=noFolderItems.map(i=>i.id) // map the array of items to an array of ids
Item.deleteDocuments(deleteIds) // actually delete the items.
```

Analogously for Items embedded on our example Actor steve we would filter out all the Items we want to delete, let’s say all weapons, map those to ids, and then pass those ids to the function:
```js
const steve=game.actors.getName("Steve") // get our actor
const weapons=steve.items.filter(i=>i.type==="weapon") // filter out all weapons
const deleteIds=weapons.map(i=>i.id) // get an array of item ids to delete
steve.deleteEmbeddedDocuments("Item",deleteIds) // delete all items with the passed ids on steve
```
## Tokens, Actors and synthetic Actors
On a final slightly more advanced note let’s talk about Tokens and their relationship with Actors.
As you may know each Token on the scene is used to represent an Actor. If you remove the associated Actor from the sidebar the Token breaks and double clicking it throws an error telling you that it can no longer find its Token. Similar to how we can double click a Token to open the associated Actor’s sheet, we can refer to the Actor associated with a Token in a macro by accessing the Actor property of the Token. So `token.actor` would give us the Actor Document associated with the Token. Now some of you may have noticed that unlinked Tokens exist, for which the Actor associated with the Token differs from the “source” Actor in the sidebar. This is what we call a synthetic Actor which has a bunch of quirks which we will go over shortly. Most of these are purely of academic interest though so should this not be of interest to you, just take away that you want to use `token.actor` to get the Actor of a relevant Token.

The Actors associated with unlinked Tokens is what we call a “synthetic Actor” because it is computed from the source Actor in the sidebar and data on the Token telling foundry what should be different from the sidebar Actor. You can find this data in the `delta` embedded Document of the Token though unless you know what you are doing you probably shouldn’t mess with it, instead updating the Actor you get from `token.actor` like usual. The reason this might be of importance to you however is that the id of that synthetic Actor is identical to the one in the sidebar, however unlike the source Actor it isn’t in the sidebar so you can’t go via `Actor.updateDocuments()` as the id you would pass it would just target the sidebar Actor. This leads to many edge cases and confusing situations, so if you ever notice that you are changing the sidebar Actor when you want a Token’s synthetic Actor, remind yourself that you want to go via `token.actor`.

## Flags
Every Document posesses a `flags` property which is intended to store arbitrary data. This is primarily used by Modules but also particularly helpful for Macros in which you wish to save some form of data between executions. The `flags` property is fundamentally identical in usage to any other property so you can update it as normal however a couple helper functions exist to enforce convention and to try and prevent you from accidentally overriding the data of others. These helper functions are `Document#setFlag(scope,key,value)` which is similar in usage to `update`, `Document#setFlag(scope,key)` which returns the relevant value or `undefined` should no such value exist and ultimately `Document#unsetFlag(scope,key)` which deletes the relevant property. Like any other change to the Document, for `setFlag()` and `unsetFlag()` to do anything the executing User needs ownership of the Document.

The `scope` of the mentioned functions is a String with valid values being the ids of enabled Modules or `"world"`. The latter is what you should use, should you wish to save arbitrary data on your Document. The `key` is likewise a String and unlike the `scope` which is more or less predetermined can be any arbitrary string of your choice. Do note that this should be unique but also note that just like with update paths you can use dot notation to set nested properties i.e. a `key` of `"foo.bar"` would result in an object of `{foo:{bar:value}}`.

### Example:
The following script if executed in a Macro will set a value of `2` on the Macro Document under `MacroDocument.flags.world.foo.bar` should no such property be set and otherwise delete the `bar` property, leaving behind an empty object at `MacroDocument.flags.world.foo`.
```js
const myData = this.getFlag("world", "foo.bar")//get the flag
if (!myData) {//if the flag doesn't exist set it
    await this.setFlag("world", "foo.bar", 2)// save the flag on the Macro Document
} else {//Otherwise show the value in a notification and remove it
    ui.notifications.notify(`Value on the Macro was:${myData}`)// Show notif
    await this.unsetFlag("world", "foo.bar")//remove value from Macro Document
}
```

# Conclusion
This concludes the introduction into basic Foundry infrastructure. If you would like to read up on how Compendiums work and how you can asynchronously fetch Documents from them please refer to the [Guide on Advanced Foundry infrastructure](https://github.com/GamerFlix/foundryvtt-api-guide/blob/main/advanced_api_guide.md). The aforementioned guide also explains the concept of Collections. 

I hope you found this guide helpful.
