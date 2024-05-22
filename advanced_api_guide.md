# Introduction
If you have not read the basic macro guide found [here](https://github.com/GamerFlix/foundryvtt-api-guide/blob/main/macro_guide.md) I strongly recommend you do so as this guide assumes that you have read it already or at least know the relevant concepts.

Like before it also recommended that you **make a test world** if you wish to follow along as you are likely to cause irreperable damage eventually while trying out things.

This guide has been updated for v12. If you find any inaccuracies please open an issue. 
While there are only minor differences a v11 version can be found at the end of [this guide](https://github.com/GamerFlix/foundryvtt-api-guide/blob/v11/macro_guide.md).

# Advanced Foundry Infrastructure

So far only Documents that are part of the World or embedded on other Documents in the World have been considered. But what of Documents in Compendiums? This brings us to the topic of Collections and Compendiums.

## Collections and Compendiums

While `Document` is the underlying Class of `JournalEntry`s, `Actor`s, `Item`s and so on, the Array-like Class holding these is often a `Collection`. The most commonly known Collections being the ones found in the `game` object (`game.actors`, `game.tables`, etc.) or those holding Embedded Documents like in the case of `Actor#items`, `Actor#effects` and `Item#effects`.
Another Collection that plays a large role is the Subclass `Pack`s also known as Compendiums.

While the various Collections of Documents found in the `game` object are loaded on log in the advantage of Packs is that the Documents they contain are only indexed on login, not fully loaded. This brings some percularities with it when trying to access Documents located within them which we will go over in the following.

### Retrieving Documents contained in Compendiums
The first step to get a Document located within a Pack is to get the Pack itself. Just having the id of the Document doesn't help us as ids are only unique within a given Collection (more on that in the section on IDs vs. UUIDs later). Since all available Packs can be found in the Collection `game.packs` this is rather trivially accomplished by using the `get`, `filter` or `find` function like for other Collections. Notably howver Packs do not have a name instead requiring you to find the id or filtering them by their label.
So if we wish to access a Pack with an id of "dnd5e.items" and a label of "Items (SRD)" we could either get it by its id:
```js
const myPack=game.packs.get("dnd5e.items")
```
or use find() and check for the title of the Pack in its metadata:
```js
const myPack=game.packs.find(i=>i.metadata.label==="Items (SRD)")
```
Once we have located the Pack we could technically iterate over it but as established earlier only the indexes of the Documents within are presently loaded. To actually get the Documents we have to call getDocuments on the Pack which asynchronously retrieves all Documents contained therein.
```js
const myPack=game.packs.get("dnd5e.items") // Get the Pack
const myDocs=await myPack.getDocuments() // Get all Documents in the Pack
```
With that done we now have a Collection of Documents in the `myDocs` variable which we can use as usual. So if we wanted to get a specific item with a name of "Abacus" we could simply call `getName` on the Collection of Documents:
```js
const myPack=game.packs.get("dnd5e.items") // Get the Pack
const myDocs=await myPack.getDocuments() // Get all Documents in the Pack
const abacus=myDocs.getName("Abacus") // The Document we wanted
```
If you read along closely you might have noticed that this gets *all* Documents in the Pack only to then actually get a single one out of that Collection which is rather inefficient. It would have been better to only retrieve the single Document we actually want. We can do that using the `Pack#getDocument()` function. This takes the id of a Document contained in the Pack to retrieve only that Document. In practice this would look like this:

```js
const myPack=game.packs.get("dnd5e.items") // get the Pack
const abacus=await myPack.getDocument("ea4xclqsksEQB1QF") // Get the Document by its id
```

Once you have retrieved the Document you can update, reference, delete, etc. it like usual. The methods only differ if you wish to batch your changes or create new Documents within the Pack, the topic of the next section.

### Things to keep in mind when editing Documents contained within a Compendium
The primary thing to keep in mind when trying to edit Documents in a Compendium is ironically the most obvious one: They are located in the Pack, not the Collection of Documents in the World. The relevance of this becomes obvious when you realize that many function such as `create()`, `createDocuments()`, `deleteDocuments()`, `updateDocuments()` and so on default to the Collection in the World. If this is the first time you hear of these function I encourage to read the earlier sections on [Batch changes](https://github.com/GamerFlix/foundryvtt-api-guide/blob/main/macro_guide.md#batch-changes) and [Creating Documents](https://github.com/GamerFlix/foundryvtt-api-guide/blob/main/macro_guide.md#creating-documents).

Changing the Collection these functions target is as simple as passing the key of the Pack that is to be modified in the second argument. So for example deleting two Items in a pack with a key of "dnd5e.items" would be:
```js
Item.deleteDocuments(
    ["id of the first document", "id of the second document"],
    {pack:"dnd5e.items"}
)
```
The process is analogous for the other mentioned functions. However it is important to keep in mind that Packs provided by the System or Modules such as the one used as an example here are often locked. While this is for good reasons (System or Module fucntions may rely on the contents, the Packs will be completly reset on update making modifications pointless) and you should generally not mess with them, there can be cases in which unlocking a locked Compendium can be desirable. A notable example may be a shared Compendium you have created and wish to modify. This brings us to the next topic, modifying the Compendium itself.

### Modifying the Compendium itself
In order to modify a Compendium you need to first retrieve it ([see previous section](https://github.com/GamerFlix/foundryvtt-api-guide/blob/main/macro_guide.md#retrieving-documents-contained-in-compendiums)) and then call `Pack#configure`. This takes an object analogue to Document#update (see this section if you forgot how that is structured). In this case we wish to modify whether or not the `Pack` is locked. 
For the purpose of this demonstration I am going to use a Compendium I just created for testing purposes with a label of "Testcompendium". So first we get the Pack by it's label using `find` and then configure it to not be locked. The relevant property for this is fittingly called `locked`.
```js
const testPack=game.packs.find(i=>i.metadata.label==="Testcompendium")
await testPack.configure({locked:false})
```
The pack is now no longer locked if it was before. In case you wish to modify some other part of the Pack (such as the ownership) log it in the console and check what is available. Do note that the metadata property is unaffected by your changes and will always hold the default state of the Pack. Another thing of note is that unlike `Document#update()` `Pack#configure()` is ***not*** a Merge operation. This means that if you intend to modify a nested property such as `ownership` the object you pass will replace the value in its entirety potentially removing ownership from even GMs if you don't explicitly include them.

### IDs and UUIDs
In the previous sections of this guide we established that in order to retrieve a relevant Document we need the Collection the Document is in aswell as some identifier to specify what Document is desired. Most commonly the identifier used for this is an ID. However if you want to get an effect, embedded on an item embedded on and actor that is located in a Compendium the amount of IDs you require quickly gets rather cumbersome. In these cases it's nice to have some identifier that is universally usable and holds all the relevant information to retrieve the Document. That identifier is the UUID or in longform: Universally Unique Identifier. Just how every Document has an id (`Document#id`), each Document also has a UUID (`Document#uuid`). In order to utilize the UUID to retrieve a given Document we can use `fromUuid()` and `fromUuidSync()`.
As the name might imply `fromUuidSync()` is synchronous while `fromUuid()` is asynchronous and as such returns a Promise that needs to be awaited. The reason you might still want to use `fromUuid()` is that unlike `fromUuidSync()` it can retrieve Documents from Compendiums which as established [here](https://github.com/GamerFlix/foundryvtt-api-guide/blob/main/macro_guide.md#retrieving-documents-contained-in-compendiums) is an asynchronous process and as such not possible to be archieved in a synchronous function.

Since these functions directly return the Document (or a Promise that resolves to a Document) using them is rather simple. As an example retrieving the Item with a UUID of "Compendium.dnd5e.items.Item.ea4xclqsksEQB1QF", and an  ID of "ea4xclqsksEQB1QF" that is located in a Pack with an ID of "dnd5e.items" can be done like follows. Either we retrieve the Pack and then the Document in two steps
```js
const myPack=game.packs.get("dnd5e.items") // get the Pack
const abacus=await myPack.getDocument("ea4xclqsksEQB1QF") // Get the Document by its id
```
or we simply its UUID like this
```js
const abacus=await fromUuid("Compendium.dnd5e.items.Item.ea4xclqsksEQB1QF")
```
If you need to retrieve the Pack for other purposes anyways the first method may be preferable but in cases where you only wish to reference a single Document uuids can be quite useful.

If you got lost during the explanation above you may want to check out [this explanation instead](https://mxzf.gitlab.io/foundry-info/documents/#document-ids).
