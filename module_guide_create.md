# This is currently under construction and will remain this way for a while. Do not rely on any of it.

# Writing a Small Module
This guide will walk you through the basic steps to set up a module aswell as introduce you to the concept of `Hooks`, the backbone of many modules. Since modules use the same FoundryVTT API that Macros do this guide requires you to have at least a basic grasp on the Foundry API as fundemental concepts like updates and Console usage will not be discussed. If you have not done so yet it is strongly recommended to check the [Macro Guide](https://github.com/GamerFlix/foundryvtt-api-guide/blob/main/macro_guide.md) before hand which goes over these basic concepts.

The guide is structured in two parts that you may wish to reference indepdently depending on how you prefer to learn. The first part concerns itself purely with scripting concepts relevant for writing a Module. On the other hand the second part concerns itself purely with getting a blank module set up that Foundry recognizes aswell as how to add Scripts written in the first part of the guide to the Module so that Foundry executes them. Since you can't really (more on that later) test your scripts without adding them to a Package you may wish to create a Module (topic of part two) before reading the first part in case you wish to immediately test the Examples presented.

On a final note it is **strongly recommended to use a test world** while developing. This was the case for Macros and is doubly true for Modules where you could create infinite loops if you are not careful. So again **use a test world**, you have been warned.

## Scripting concepts relevant for writing a Module
Before we get into the basic concepts I would once again like to urge you to **use a test world** if you wish to follow along the guide.

The fundemental difference between code attached to a Package such as a System, World or Module and a Macro is that the code in a Macro only gets executed when you execute the Macro. Code attached to a Package on the other hand gets executed when you log into a World while the Package is enabled. For sake of brevity and since the guide is primarily about writing a Module I will refer to code attached to a Package as Module code.

Module code being executed on startup has multiple effects that need to be taken into account. First and foremost it means that the code gets executed on each client that logs in. This is an important difference to a Macro since the Macro code will only be executed on the Client calling the Macro. The other thing to keep in mind is that at the moment the module code is executed not many things will have been loaded. As such you will usually need to postpone the execution of certain code portions until the relevant thing is available.

This is where we get into the main topic of this guide: `Hooks`. 

### Basics of Hooks

`Hooks` fundamental concept is to provide the ability to "react" to things happening. Be that as simple as Foundry finishing certain parts of its initilization ("init", "i18init", "setup" and "ready") or as specific as the combat turn changing ("combatTurn"). Because hooks allow you to react to various things happening many systems and some modules add their own hooks aswell to allow you to react or otherwise modify actions caused by these Packages. It is important to note however that `Hooks` are fundamentally synchronous. This means that as long as you register a synchronous function that function will executed in its entirety before the code calling the hooked functions continues. Passing an asynchronous function, for example on in which you await a Document update, will not cause the code to wait. This will be of relevance in the later pre and post Hooks section of this guide but is important to keep in mind.

--------------------------
- Examples here
**Potential examples:**
```javascript
Hooks.on("renderChatMessage", function(message, [html]) {
  html.querySelector(".dice-tooltip")?.classList.toggle("expanded", true);
});
```
- Alternatively play sound on user log in
- add sounds for a door
-------------
- Remember to refresh for changes to take effect and check the relevant guide later



### Hooks and You: Finding the relevant hook
While the previous examples work well for their purpose and introduce a few hooks they obviously don't showcase all of them. Doing so would also be somewhat of a fools errand due to the sheer number of them. This brings up the question of how you would find the specific hook you want for your purpose.
One method would be to scour the Foundry API docs as all core hooks are listed there. The downside of this is that the docs are quite dense and on top of that only provide core Foundry hooks. Any hooks your modules or systems might add are not listed.

This is where we turn our attention to the console once again. Simply setting CONFIG.debug.hooks to true, will cause all hooks and their arguments to be logged in the console as they are called. Therefore the best course of action when trying to find the relevant hook for a specific action you wish to "react" is to:
1. Set `CONFIG.debug.hooks=true`
2. Do the action you want to find the hook for
3. Set back to false (`CONFIG.debug.hooks=false`)

Setting the property back to false to stop the logging might seem insignificant but is rather important. The reason for this  becomes apparent when you notice just how many hooks are constantly being called. If you don't set it back to false what you are looking for might just get drowned out by all the log spam. As an example here's what happens when I do XYZ and then move my cursor back over to the console to refocus the input field.

**XYZ PICTURE OF CONSOLE BEING ABSOLUTELY SPAMMED BY THE ACTIONS DESCRIBED ABOVE**

### pre and post hooks
**Pre hooks**
- react to thing before it actually happened
- cancel thing
- Only executing client

**Post hooks**
- Just reacting to thing happening
- No cancel possible
- All clients
**Examples:**
- macro folder author thingie

## Setting up a module

### Creating the module 
In order to have a module to attach scripts to you first need to create it. Should you already have one for sharing Compendiums between worlds you can use that one though it doesn't hurt to make a second one to be able to deactivate all scripts without removing your Compendiums. Luckily Foundry takes care of the creation for you with an easy to use GUI. To access it go to the setup page. and click the little cog Icon next to "Update All" on the Add-on Modules tab:

![create_module](https://github.com/user-attachments/assets/c4e5da14-225b-4d2d-b220-dcb43468c322)

On the next page you can fill in various things like the module title that is later displayed in your manage modules list. You don't need to fill in everything and the default values are largely fine as you can edit them later if you wish to do so. There is however **one exception** to this. And that is the identifier of the Package. **The identifier can not be changed later** (without breaking everything depending on it) so try to think of one that is unlikely to collide with anything else.

![Id_is_important](https://github.com/user-attachments/assets/3549317c-4c78-432e-9e77-c35c8fc35c49)

Once you are happy with your choices hit create Module and the Module will appear in your list. Now that you have your own Module shutdown Foundry and proceed to the next step.

### Adding scripts to a package
To attach a script to a Module it is crucical that you first **shutdown Foundry** if you haven't done so yet. This is of vital importance as otherwise any changes you attempt to make to your Module's Manifest will not save.
After you shutdown Foundry navigate to your userdata Folder. The *default* locations depend on your OS. These are:
Windows 10/11 - %localappdata%\FoundryVTT\
macOS - ~/Library/Application Support/FoundryVTT
Linux - ~/.local/share/FoundryVTT

Once you have found your userdata navigate to the `modules` folder and then the folder of your module, the name of which is identical to the identifier you chose earlier:
![find module](https://github.com/user-attachments/assets/7159a36b-e740-4426-b19e-4899473c48c9)
Open the folder and then edit the Manifest. Since this is a Module it'll be called `module.json`. It should look something like this though it may have more fields if you also have Compendium Packs or dependencies specified.
![grafik](https://github.com/user-attachments/assets/87180019-6f41-406b-8cd0-f62e614cd303)
Add a new line **in the root of the object** so directly inside the brackets on the first and last line **not** inside another set of brackets therein. Add a key of `"esmodules"` with a value of an Array in there and keep an eye on the commas. They should be between each element but not after the last on of each Object/Array. An example can be seen here:
![grafik](https://github.com/user-attachments/assets/3338bd58-1aec-4f8d-bbf4-6783f9024146)
Now in your modules folder createa a new text file and give it a name of your choice. Change the extension to `.js`. If you are on Windows you may need to enable showing those first a tutorial to which can be found [here](https://support.microsoft.com/en-us/windows/common-file-name-extensions-in-windows-da4a4430-8e76-89c5-59f7-1cdbbc75cb01).
![grafik](https://github.com/user-attachments/assets/3c61ddbe-c16d-4428-b185-642ee881f69a)
This will be the file you post all your code in which you can do now. Back in the manifest add the filename to your Array. In my case I called it `"myScript.js"`(see picture above) so that will be what I write in the Array:
![grafik](https://github.com/user-attachments/assets/51a4e0f2-f9ea-4627-be15-db310de690f0)
**Save the file then close it.** Open Foundry back up and check if the module still appears in your list. If not you likely have an errant comma or similar in your Manifest. In this case I recommend using an IDE such as [VSCode](https://code.visualstudio.com/) or an [online JSON linter](https://jsonlint.com/) to find the issue. You can also always ask for additional assistance on the [official FoundryVTT Discord server](https://discord.com/invite/foundryvtt)
If everything appears fine and you haven't added your code to the `.js` file yet do so now and save it. In order for this to take effect you will need to refresh though, for edits of `.js` files you do not need to fully restart as that is only nessecary when editing the manifest.

Now you have your very own Module complete with a script.
