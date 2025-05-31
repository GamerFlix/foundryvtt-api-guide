# Introduction
This guide will walk you through how to make your Macros or Scripts work with the Scene region behaviours of "Execute Macro" or "Execute Script" respectively. In doing so it however assumes that you already know how to accomplish the relevant task in a normal Macro or Script. If this is not the case please refer to [this guide which introduces you to basic Foundry infrastructure](https://github.com/GamerFlix/foundryvtt-api-guide/blob/main/macro_guide.md)

For general information on Regions refer to the [Official Knowledge Base article on Regions](https://foundryvtt.com/article/scene-regions/). I will not cover how to add Behaviors or Events in general.


# Difference between macro and script behaviors
While the Script and Macro behaviours may look identical at first glance there are three key differences I'll go over in this chapter. 

Most notably Macro behaviors still require the executing user to have permissions to execute the relevant Macro Document and by default they are only executed by a single User with sufficient permissions though this can be changed by ticking a checkbox (see image below). 

![macro_behavior_checkbox](https://github.com/user-attachments/assets/1ac33cda-1622-43c4-a1b9-5ceac15acb13)

Script behaviors on the other hand are not limited by any permissions and always execute on all connected clients. This can be quite useful in cases where you want to trigger actions on other clients which would normally require a module.In many cases this however means that you will have to ensure that your code that actually does something is only executed where needed, and most importantly only once in most cases, via additional lines of code near the start of the script. More on that later. 

The third and final difference is that Macro and Script Behaviors have access to slightly different arguments. While Macro Behaviors have access to all the arguments Regions do, they also have the usual Macro arguments of speaker, actor, token, character and scope. In addition to the region provided arguments of `scene`, `region`, `behavior` and `event`. It should be noted that all extra arguments a Macro Behavior receives are derived from the same data a Script Behavior receives. This means there is functionally no difference in the data available to you in a Script Behavior compared to a Macro Behavior (this was verified by looking at the source code of FoundryVTT 12.331). This also means that while in a Macro executed from the Hotbar the `token` argument is always the first selected Token, in a Macro Behavior triggered by for example a "Token Enters" event it is the entering Token regardless of what clients the code is executed on.

For now let's look at making sure the script only runs on the desired clients.

# Ensuring the script executes only once
You generally want to execute your script either on the client that is causing the behavior to trigger or the gm client. In the case of the former you can simply set the aforementioned checkbox in the Region for Macro behaviors. However for script behaviors we don't have that luxury. As such you'll need to add an early return at the start of the script like this:
```js
if (game.user!==event.user) return
```
This would ensure that any line after this one only gets executed on the client that triggered the behavior. This is useful if you wish to make specific changes that the user can do, such as change the token elevation. If it were to execute on all clients every client that can would increase the elevation leading to higher changes than you want while also spamming the server with requests and any user without permission would get an error, all around not a good experience.

However in other cases you want to execute on a single gm client. This is usually nessecary when you want to make changes to something only a GM user has access to, such as changing tiles, lights or playlists. Because there is no preexisting checkbox to ensure this on the Macro behavior it has to use an early return just like the script behavior which would look like this:
```js
if (game.user!==game.users.activeGM) return
```
Now you may have noticed that we check for the active gm, not just whether the current user is a gm user. This is to ensure that it works fine even in games with multiple gm users. You could technically forgoe the previously suggested option of only triggering on the executing user in favor of always deferring to the gm user but this puts unnessecary stress on the gm user which can pile up if every script and module does this simultaneously. As such executing on the triggering client is generally preferable if possible.

Now with that out of the way let's take a closer look at what arguments script and macro behaviors are provided.

# Arguments available to Regions
Since putting a complete list of arguments in this writeup would take a long time of triggering, taking screenshots, and verifying this has not been done yet in favor of actually publishing this guide. That being said there are a couple importants points to go over here. The first is how to even check what arguments are available. This is doubly important in case you have modules or your system modifying regions.
In order to see what arguments are passed to a relevant script or macro you will simply want to put the following line into the code:
```js
console.log(arguments)
```
To quote [mdn web docs]("https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/arguments"):

> `arguments` is an array-like object accessible inside functions that contains the values of the arguments passed to that function."

This means that all variables passed directly to the function can be accessed in this object, therefore giving us a complete overview as to what the region provides our script with. Logging it looks something like this once expanded:

![image_of_arguments](https://github.com/user-attachments/assets/d65b5f05-b301-4ea3-9371-745e425b3445)

While this may seem a little overwhelming at first it isn't too bad once you realize that the line marked in the image above tells you what the names of all the passed variables are. I also color-coded the variables so you can see the difference between Script (top image) and Macro (bottom image) Behaviors. If we use this knowledge to annotate the image for clarity it becomes quite clear what argument is what:

![annotated_arguments_cut](https://github.com/user-attachments/assets/e1898201-c083-4d35-8b82-2bf10159e21a)

Most of the time you will want to look at the `event` argument. Not only does it hold information about what Event triggered the behavior but `event.data` holds information about triggered the Event e.g. the Document of the Token that entered a Region which would be `event.data.token` in case of the "Token Enters" Event.

# Example scripts
With that out of the way let's look at some example scripts. Where relevant these will be seperated into
- a codeblock to ensure execution only on one end / set up instructions
- a codeblock following after regardless

You will notice that the first codeblock ensuring execution by only one user is identical between some examples. This is intentional as those examples require the same considerations. I decided against simply referring to a "master" example elsewhere. This is so that people who wish to only copy relevant code snippets for their game have all the nessecary configuration steps within one topic and don't need to read the whole guide.
## Open character sheet
To ensure that only the triggering user's character sheet is opened ensure that the the check box in the Macro Behavior config is **unchecked**. In case of a Script Behavior add the following line at the top of the script:
```js
if (game.user!==event.user) return
```
Main codeblock you need:
```js
game.user.character.sheet.render(true) //Open current users character sheet
```

## Open the sheet of a specific actor
To ensure that only the triggering user's character sheet is opened ensure that the the check box in the Macro Behavior config is **unchecked**. In case of a Script Behavior add the following line at the top of the script:
```js
if (game.user!==event.user) return
```
Main codeblock you need:
```js
game.actors.getName("name of the actor").sheet.render(true) //Open specific actor's sheet in the world
```

## Open a specific Journal Entry
To ensure that only the triggering user's character sheet is opened ensure that the the check box in the Macro Behavior config is **unchecked**. In case of a Script Behavior add the following line at the top of the script:
```js
if (game.user!==event.user) return
```
Main codeblock you need:
```js
game.journal.getName("name of journal").sheet.render(true)//open speciifc JE
```

## Open a specific Journal Page
To ensure that only the triggering user's character sheet is opened ensure that the the check box in the Macro Behavior config is **unchecked**. In case of a Script Behavior add the following line at the top of the script:
```js
if (game.user!==event.user) return
```
Main codeblock you need:
```js
const page = game.journal.getName("name of journal").pages.getName("name of page");page.parent.sheet.render(true, {pageId: page.id});//open specific page in specific journal
```

## View a specific Scene
To ensure that only the triggering user's character sheet is opened ensure that the the check box in the Macro Behavior config is **unchecked**. In case of a Script Behavior add the following line at the top of the script:
```js
if (game.user!==event.user) return
```
Main codeblock you need:
```js
game.scenes.getName("name of scene").view()//change to specific scene
```

## Activate a specific scene
Since only a GM can activate Scene's the first line ensures execution by only one GM. If you are using a Macro Behavior make sure you **check** the checkbox in the Macro Behavior config. Region Behaviors require no additional adjustments.
```js
if (game.user!==game.users.activeGM) return
game.scenes.getName("name of scene").activate()//activate speciifc scene
```

## Increase the elevation of a triggering Token
To ensure that only the triggering user's character sheet is opened ensure that the the check box in the Macro Behavior config is **unchecked**. In case of a Script Behavior add the following line at the top of the script:
```js
if (game.user!==event.user) return
```
Main codeblock you need:
```js
event.data.token.update({elevation:event.data.token.elevation+10})
```
This will incrase the elevation of a triggering Token by 10. Adjust the 10 as nessecary.

## Set the elevation of a triggering Token
To ensure that only the triggering user's character sheet is opened ensure that the the check box in the Macro Behavior config is **unchecked**. In case of a Script Behavior add the following line at the top of the script:
```js
if (game.user!==event.user) return
```
Main codeblock you need:
```js
event.data.token.update({elevation:10})
```
This will set the elevation of a triggering Token to 10. Adjust the 10 as nessecary.

## Requesting additional examples

If you need additional examples for scripts that are not longer than 2-5 lines of code feel free to make an issue.

## empty example for future copying
code block only for scripts if applicable
```js
```
Main codeblock you need:
```js

```
