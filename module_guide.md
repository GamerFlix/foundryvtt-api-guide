# This is currently under construction and will remain this way for a while. Do not rely on any of it.

# Writing a Small Module
This guide will walk you through the basic steps to set up a module aswell as introduce you to the concept of `Hooks`, the backbone of many modules. Since modules use the same FoundryVTT API that Macros do this guide requires you to have at least a basic grasp on the Foundry API as fundemental concepts like updates and Console usage will not be discussed. If you have not done so yet it is strongly recommended to check the [Macro Guide](https://github.com/GamerFlix/foundryvtt-api-guide/blob/main/macro_guide.md) before hand which goes over these basic concepts.

The guide is structured in two parts that you may wish to reference indepdently depending on how you prefer to learn. The first part concerns itself purely with scripting concepts relevant for writing a Module. On the other hand the second part concerns itself purely with getting a blank module set up that Foundry recognizes aswell as how to add Scripts written in the first part of the guide to the Module so that Foundry executes them. Since you can't really (more on that later) test your scripts without adding them to a Package you may wish to create a Module (topic of part two) before reading the first part in case you wish to immediately test the Examples presented.

On a final note it is **strongly recommended to use a test world** while developing. This was the case for Macros and is doubly true for Modules where you could create infinite loops if you are not careful. So again **use a test world**, you have been warned.

## Scripting concepts relevant for writing a Module
Before we get into the basic concepts I would once again like to urge you to **use a test world** if you wish to follow along the guide.

- *why* you want a world script / module and why executing as a macro is not enough
- Macro executed only on one client
- Module code gets executed during startup
- If something needs to happen on multiple clients -> Module
- But if it gets executed at startup how do you react to X, only execute when x happens?
- Hooks, next section:
- Reloading to take effect

### Hooks and You: Finding them in the console
- CONFIG.debug.hooks=true
- Screenshot
- Total spam
Partial Prewrite:
While scouring the API docs for available hooks is certainly a possibility the docs are usually dense and hard to comprehend. Furthermore they don't include any hooks your system or potential modules added. As such it is advisable to simply check the console. In order to do so set CONFIG.debug.hooks to true, do the relevant action you are looking for the hook for, and then set it back to false immediately after. This will log the hooks and their arguments in the console as they are called. Setting the property back to false to stop the logging might seem insignificant but is rather important. The reason for this  becomes apparent when you notice just how many hooks are constantly being called. If you don't set it back to false what you are looking for might just get drowned out by all the log spam.

- Enable, do thing that's needed, disable
- Search for relevant thing
```javascript
Hooks.on("renderChatMessage", function(message, [html]) {
  html.querySelector(".dice-tooltip")?.classList.toggle("expanded", true);
});
```
- Alternatively play sound on user log in
- add sounds for a door
### pre and post hooks
- macro folder author thingie

## Setting up a module

### Creating the module 

### Adding scripts to a package (world script and module)
- SHUTDOWN FOUNDRY
- What gets cleared on reload

