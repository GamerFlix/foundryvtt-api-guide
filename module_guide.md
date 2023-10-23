# This is currently under construction and will remain this way for a while. Do not rely on any of it.

# Writing a Small Module
- Walk through creating small module / worldscript
- explain how there's little difference
- Explain hooks and usage of hooks
- Recommend reading macro guide first if not familiar with foundry api already

## Preamble
- *why* you want a world script / module and why executing as a macro is not enough
- Macro executed only on one client
- Module code gets executed during startup
- If something needs to happen on multiple clients -> Module
- But if it gets executed at startup how do you react to X, only execute when x happens?
- Hooks, next section:

## Hooks and You: Finding them in the console
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
## pre and post hooks
- macro folder author thingie

## Creating the module 

# Adding scripts to a package (world script and module)
