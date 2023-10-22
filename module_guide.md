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
- Enable, do thing that's needed, disable
- Search for relevant thing
```javascript
Hooks.on("renderChatMessage", function(message, [html]) {
  html.querySelector(".dice-tooltip")?.classList.toggle("expanded", true);
});
```

- Alternatively play sound on user log in
## pre and post hooks
- macro folder author thingie

## Creating the module 

# Adding scripts to a package (world script and module)
