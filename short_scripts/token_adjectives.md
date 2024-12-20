# Work in progress, do not rely on this!

# Adjusting available Token adjectives
This guide will take you through the process of adding, changing and removing adjectives from the list used for the "Prepend Adjetive to Name of Unlinked Tokens?" option in the Protoype Token config.

## Adding new adjectives

The adjectives Foundry uses for the aforementioned option are informed from the localization files available to Foundry. The keys for these values follow the structure of `"TOKEN.Adjectives.uniqueIdenfierHere"`. For better visulization and excerpt from the `en.json` shipped with the core Software can be seen below:
```js
"TOKEN.Adjectives.overwhelmed": "Overwhelmed",
"TOKEN.Adjectives.painful": "Painful",
"TOKEN.Adjectives.panicked": "Panicked",
"TOKEN.Adjectives.pensive": "Pensive",
"TOKEN.Adjectives.perplexed": "Perplexed",
"TOKEN.Adjectives.pessimistic": "Pessimistic",
"TOKEN.Adjectives.proud": "Proud",
"TOKEN.Adjectives.puzzled": "Puzzled",
"TOKEN.Adjectives.relieved": "Relieved",
"TOKEN.Adjectives.remorseful": "Remorseful",
"TOKEN.Adjectives.resentful": "Resentful",
"TOKEN.Adjectives.restless": "Restless",
"TOKEN.Adjectives.sad": "Sad",
"TOKEN.Adjectives.satisfied": "Satisfied",
"TOKEN.Adjectives.scared": "Scared",
"TOKEN.Adjectives.selfish": "Selfish",
"TOKEN.Adjectives.shocked": "Shocked",
"TOKEN.Adjectives.shy": "Shy",
"TOKEN.Adjectives.silly": "Silly",
"TOKEN.Adjectives.sleepy": "Sleepy",
```
As such adding additional adjectives requires you to:
1. Create a Module (see [here](https://github.com/GamerFlix/foundryvtt-api-guide/blob/main/module_guide_create.md) for instructions if nesssecary)
2. Create a localization file with a similar structure (more information on that later)
3. Attach the file to the manifest of your module

I went over how to create a module using Foundry's built-in Module creator [here](https://github.com/GamerFlix/foundryvtt-api-guide/blob/main/module_guide_create.md) so if you don't have a module yet to use as a basis plese refer to the aforementioned guide.

Once you have a module you will want to create a new .json file inside your Module's folder, as with adding script it is of importance that the file extension is correct here which may be difficult should your OS hide them. Instructions on enabling to show file extensions on Windows can be found [here](https://support.microsoft.com/en-us/windows/common-file-name-extensions-in-windows-da4a4430-8e76-89c5-59f7-1cdbbc75cb01).
For the English language your json file should be called `en.json` for other languages you can find 

https://foundryvtt.com/article/localization/

## Changing existing adjectives

## Removing adjectives from the list