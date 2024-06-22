# Introduction

This guide is currently under construction and should not be relied upon

This guide will walk you through how to make your Macros or Scripts work with the Scene region behaviours of "Execute Macro" or "Execute Script" respectively. In doing so it however assumes that you already know how to accomplish the relevant task in a normal Macro or Script. If this is not the case please refer to [this guide which introduces you to basic Foundry infrastructure](https://github.com/GamerFlix/foundryvtt-api-guide/blob/main/macro_guide.md)

For general information on Regions refer to the Official Knowledge Base article on Regions TO BE ADDED or this video tutorial by alaustin here TO BE ADDED. I will not cover how to add behaviors or events in general.


# Difference between macro and script behaviors
While the Script and Macro behaviours may look identical at first glance their are three key differences I'll go over in this chapter. 

Most notably Macro behaviors still require the executing user to have permissions to exdcute the relevant Macro Document. This may be obvious to some but since by default Macro behaviours trigger on every client when triggered, not just the one causing the triggering action, this can lead to a spam of permission errors. As such you'll want to make sure all clients you inticipate to execute the macro have permission to do so. 

The second difference is that while Macro behaviours allow you to set an option that they are only executed on the User causing the triggering action. Script behaviors possesss no such option and as such you will have to ensure that your action is only executed where needed, and most importantly only once in most cases, via additional lines of code near the start of the script. More on that later. 
The third and final difference is that Macros and Scripts have access to slightly different arguments. While Macros have access to all the arguments regions do, they also have the usual Macro arguments of speaker, actor, token, character and scope. In addition to the region provided arugments of scene, region, behavior and event. What arguments are available exactly will be the topic of a later chapter. For now let's look at making sure the script only runs on the desired clients.

# Ensuring the script executes only once
You generally want to execute your script either on the client that is causing the behavior to trigger or the gm client. In the case of the former you can simply set the following checkbox in the region for Macro behaviors:
IMAGE HERE
However for script behaviors we don't have that luxury. As such you'll need to add an early return at the start of the script like this:
```js
if (game.user!==event.user) return
```
This would ensure that any line after this one only gets executed on the client that triggered the behavior. This is useful if you wish to make specific changes that the user can do, such as change the token elevation. If it were to execute on all clients every client that can would increase the elevation leading to higher changes than you want while also spamming the server with requests and any user without permission would get an error, all around not a good experience.

However in other cases you want to execute on a single gm client. This is usually nessecary when you want to make changes to something only a GM user has access to, such as changing tiles, lights or playlists. Because there is no preexisting checkbox to ensure this on the Macro behavior it has to use an early return just like the script behavior which would look like this:
```js
if (game.user!==game.users.activeGM) return
```
Now you may have noticed that we check for the active gm, not just whether the current user is a gm user. This is to ensure that it works fine even in games with multiple gm users. You could technically forgoe the previously suggested option of only triggering on the executing user in favor of always deferring to the gm user but this puts unnessecary stress on the gm user which can pile up if every script and module does this simultaneously. As such executing on the triggering client is generally preferable if possible.

Now with that out of the way let's take a look at what arguments script and macro behaviors are provided.

# Arguments available to Regions
Since putting a complete list of arguments in the middle of this writeup would require lots of scrolling to get to the examples it can instead be found at the end of the guide. That being said there are a couple importants points to go over here. The first is how to even check what arguments are available. This is doubly important in case you have modules or your system modifying regions.
In order to see what arguments are passed to a relevant script or macro you will simply want to put the following line into the code:
```js
console.log(arugments)
```
To quote [mdn web docs]("https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments"):

> `arguments` is an array-like object accessible inside functions that contains the values of the arguments passed to that function."

This means that all variables passed directly to the function can be accessed in this object, therefore giving us a complete overview as to what the region provides our script with. Logging it looks something like this once expanded:

IMAGE HERE

While this may seem a little overwhelming at first it isn't too bad once you realize that the line marked in the image above tells you what the names of all the passed variables are. So if we annotate the image for clarity it becomes quite clear what argument is what:

IMAGE HERE

Now as mentioned earlier Macros get their normal arguments in addition to what regions provide 

- Verify that the token is still the controlled token not the triggering the behavior if applicable
# Example scripts
With that out of the way let's look at some example scripts. Where relevant these will be seperated into
- codeblock to ensure execution only on one end / set up instructions
- codeblock following after regardless

You will notice that the first codeblock ensuring execution by only one user is identical between some examples. This is intentional as those examples require the same considerations. I decided against simply referring to a "master" example elsewhere so that people who simply just to the relevant examples for use in their game can easily see all the nessecary configuration within one topic.
## Open character sheet
To ensure that only the triggering user's character sheet is opened set the check box in the Macro Behavior config or in case of a Script Behavior add the following line at the top of the script:
```js
if (game.user!==event.user) return
```
Main codeblock you need:
```js
game.user.character.sheet.render(true) //Open current users character sheet
```

## Open the sheet of a specific actor
To ensure that only the triggering user's character sheet is opened set the check box in the Macro Behavior config or in case of a Script Behavior add the following line at the top of the script:
```js
if (game.user!==event.user) return
```
Main codeblock you need:
```js
game.actors.getName("name of the actor").sheet.render(true) //Open specific actor's sheet in the world
```

## Open a specific Journal Entry
To ensure that only the triggering user's character sheet is opened set the check box in the Macro Behavior config or in case of a Script Behavior add the following line at the top of the script:
```js
if (game.user!==event.user) return
```
Main codeblock you need:
```js
game.journal.getName("name of journal").sheet.render(true)//open speciifc JE
```

## Open a specific Journal Page
To ensure that only the triggering user's character sheet is opened set the check box in the Macro Behavior config or in case of a Script Behavior add the following line at the top of the script:
```js
if (game.user!==event.user) return
```
Main codeblock you need:
```js
const page = game.journal.getName("name of journal").pages.getName("name of page");page.parent.sheet.render(true, {pageId: page.id});//open specific page in specific journal
```

## View a specific Scene
To ensure that only the triggering user's character sheet is opened set the check box in the Macro Behavior config or in case of a Script Behavior add the following line at the top of the script:
```js
if (game.user!==event.user) return
```
Main codeblock you need:
```js
game.scenes.getName("name of scene").view()//change to specific scene
```

## Activate a specific scene
Since only a GM can activate Scene's the first line ensures execution by only one GM:
```js
if (game.user!==game.users.activeGM) return
game.scenes.getName("name of scene").activate()//activate speciifc scene
```

## Modify token size
TBA once I figure out what token to refer to

code block only for scripts if applicable
```js
```
Main codeblock you need:
```js

```

## example
code block only for scripts if applicable
```js
```
Main codeblock you need:
```js

```
## example
code block only for scripts if applicable
```js
```
Main codeblock you need:
```js

```
## example
code block only for scripts if applicable
```js
```
Main codeblock you need:
```js

```
## example
code block only for scripts if applicable
```js
```
Main codeblock you need:
```js

```

# Full list of arguments
