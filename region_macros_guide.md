# Introduction
This guide will walk you through how to make your Macros or Scripts work with the Scene region behaviours of "Execute Macro" or "Execute Script" respectively. In doing so it however assumes that you already know how to accomplish the relevant task in a normal Macro or Script. If this is not the case please refer to [this guide which introduces you to basic Foundry infrastructure](https://github.com/GamerFlix/foundryvtt-api-guide/blob/main/macro_guide.md)

For general information on Regions refer to the [Official Knowledge Base article on Regions](https://foundryvtt.com/article/scene-regions/). I will not cover how to add Behaviors or Events in general.


# Difference between macro and script behaviors
While the script and macro behaviours may look identical at first glance there are three key differences I'll go over in this chapter. 

## Macro behaviours
Most notably macro behaviors still require the executing user to have permissions to execute the relevant macro document and by default they are only executed by a single user with sufficient permissions though this can be changed by ticking a checkbox (see image below). 

![macro_behavior_checkbox](https://github.com/user-attachments/assets/1ac33cda-1622-43c4-a1b9-5ceac15acb13)

## Script behaviours
Script behaviors on the other hand are not limited by any permissions and always execute on all connected clients. This can be quite useful in cases where you want to trigger actions on other clients which would normally require a module.In many cases this however means that you will have to ensure that your code that actually does something is only executed where needed, and most importantly only once in most cases, via additional lines of code near the start of the script. More on that later. 

## Arguments access for script and macro
The third and final difference is that macro and script behaviors have access to slightly different arguments.
In addition to the region provided arguments of
- scene
- region
- behavior
- event

Macro behaviors have access to all the arguments regions do and they also have the usual macro arguments of:
- speaker
- actor
- token
- character
- scope

It should be noted that all extra arguments a macro behavior receives are derived from the same data source script behavior receives. This means there is functionally no difference in the data available to you in a script behavior compared to a macro behavior (this was verified by looking at the source code of FoundryVTT 12.331).   
This means that for a regular macro executed from the hotbar which uses the `token` argument the token is always the first selected token, in a macro behavior triggered by for example by a "Token Enters" event, it is the entering token regardless of what clients the code is executed on.

For now let's look at making sure the script only runs on the desired clients.

# Ensuring the code of the script executes only on the desired client
You generally want to execute your script either on the client that is causing the behavior to trigger or the gm client. In the case of the former you can simply set the aforementioned checkbox in the region for macro behaviors. However for script behaviors we don't have that luxury. As such you'll need to add an early return at the start of the script like this:
```js
if (game.user!==event.user) return
```
This would ensure that any line after this only gets executed on the client that triggered the behavior. This is useful if you wish to make specific changes that the user is allowed to do, such as change the token elevation. If it were to execute on all clients every client would try to increase the elevation leading to unecessary load while also spamming the server with requests and any user without the necessary permission would get an error, all around not a good experience.

However there are other cases where you only want to execute on a single gm client. This is usually necessary when you want to make changes to something only a GM user has access to, such as changing tiles, lights or playlists. Because there is also no preexisting checkbox to ensure this on the macro behavior tab, it makes sense to use an early return just like for the script behavior which would look like this:
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

Most of the time you will want to look at the `event` argument. Not only does it hold information about what Event triggered the behavior but `event.data` also holds information about what triggered the Event e.g. the Document of the Token that entered a Region which would be `event.data.token` in case of the "Token Enters" Event.

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
To ensure that the Journal Entry is only opened for the triggering User ensure that the the check box in the Macro Behavior config is **unchecked**. In case of a Script Behavior add the following line at the top of the script:
```js
if (game.user!==event.user) return
```
Main codeblock you need:
```js
game.journal.getName("name of journal").sheet.render(true)//open speciifc JE
```

## Open a specific Journal Page
To ensure that the Journal Entry Page is only opened for the triggering User ensure that the the check box in the Macro Behavior config is **unchecked**. In case of a Script Behavior add the following line at the top of the script:
```js
if (game.user!==event.user) return
```
Main codeblock you need:
```js
const page = game.journal.getName("name of journal").pages.getName("name of page");page.parent.sheet.render(true, {pageId: page.id});//open specific page in specific journal
```

## View a specific Scene
To ensure that the Scene is only viewed for the triggering User ensure that the the check box in the Macro Behavior config is **unchecked**. In case of a Script Behavior add the following line at the top of the script:
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
To ensure that only the triggering User tries to update the Token is opened ensure that the the check box in the Macro Behavior config is **unchecked**. In case of a Script Behavior add the following line at the top of the script:
```js
if (game.user!==event.user) return
```
Main codeblock you need:
```js
event.data.token.update({elevation:event.data.token.elevation+10})
```
This will incrase the elevation of a triggering Token by 10. Adjust the 10 as nessecary.

## Set the elevation of a triggering Token
To ensure that only the triggering User tries to update the Token is opened ensure that the the check box in the Macro Behavior config is **unchecked**. In case of a Script Behavior add the following line at the top of the script:
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
