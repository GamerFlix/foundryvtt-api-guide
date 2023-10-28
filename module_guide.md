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

### Hooks and You: Finding them in the console

`Hooks` allow you to "react" to things happening

- CONFIG.debug.hooks=true
- Screenshot
- Total spam

- Reloading to take effect
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


- Order:
init
i18init
setup
ready
### pre and post hooks
- macro folder author thingie

## Setting up a module

### Creating the module 

### Adding scripts to a package (world script and module)
- SHUTDOWN FOUNDRY
- What gets cleared on reload

