# Introduction

In this guide we'll go over the code nessecary to create a small Module that adjusts the maximum zoom of a scene. For a guide on actually making the Module itself, not the code, please see the [guide on creating a Module](https://github.com/GamerFlix/foundryvtt-api-guide/blob/main/module_guide_create.md).

## Hooks and adjusting the CONFIG

Many aspects of Foundry can be adjusted by simply making changes to the global `CONFIG` object. Among the ability of registering new door sounds and making changes to system behaviour (if supported by the system) this also includes the goal of this Module: Changing the maximum zoom level.
Either by digging around in the console, asking somebody else who has done so or looking at the source code you will come to find that the maximum zoom level is controlled by the value set in  `CONFIG.Canvas.maxZoom`.

While we could simply assign a new value to this in a Macro and be done, this would need to be manually reapplied after each reload which is why a Module that sets it for us is much more suitable.

However since scripts and esmodules attached to a Module are executed way before the CONFIG object exists we'll need to register a function on a suitable Hook that changes the value for us. For more indepth information on Hooks see this wiki article but for our purposes the "ready" hook, which is called at the end of the initialization, will suffice.
As such all you need to do if you wish to set the zoom level to a fixed value would be the following code:

```javascript
Hooks.once("ready",()=>{
CONFIG.canvas.maxZoom=20;//Or whatever value you want
})
```

## Registering a setting

While this is all well and good opening the script every time we want to adjust the maximum zoom level is quite bothersome. Instead we could register a Setting which can then be configured via the Configure Setting menu like any other setting.

To do so we add another block of code that gets executed during the "init" Hook so that the setting is registered in time.
We do this with the following snippet. Please make sure to read the comments as they explain the reasoning behind some of the values and alternative decisions you could make. For example using a client setting would allow everyone to set their own max zoom level at the cost of it being browser bound and as such the value being lost when you delete local storage.

```javascript
Hooks.once("init",()={
game.settings.register('the-id-of-your-module-goes-here', 'maximumZoom', {//maximumZoom is the id of the setting, we'll need this later to get the value of it!
  name: 'Maximum Zoom',//just a name could be called whatever you want
  hint: 'Sets the maximum zoom level you can zoom out to on the canvas.',//explanation text, likewise whatever you want
  scope: 'world',     // "world" means only the GM can adjust and its the same for everyone, "client" means everyone has to adjust it themselves AND it's browser(client) bound
  config: true,       // If false it won't appear in the configure settings page
  type: Number,//we need a number
  default: 20,//Default value
  onChange: value => { // value is the new value of the setting
 CONFIG.Canvas.maxZoom=value;
  },
  requiresReload: true,//since we reapply the setting onChange we don't need to reload
  range: {
    min: 0//no negative max zoom
  },
});
})
```
Do note that you'll need to adjust the id of your module in the code above to be identical to the one you chose during the Module creation process.

Now that we have registered a setting we'll need to actually use it, rather than our fixed number, to set the max zoom in our setup hook. Otherwise the CONFIG would only be adjusted when you change the setting due to the onChange function.

To do so we simply get the value of the setting and use that rather than a fixed number:
```javascript
Hooks.once("ready",()=>{
CONFIG.Canvas.maxZoom=game.settings.get("your-module-id-here","maximumZoom");//module id and setting id respectively
})
```

And just like that we have all we need to adjust the maximum zoom on the fly! All that remains now is to attach the code to a Module, if you haven't done so already, and configure the setting to your liking. As a reminder you can find a guide on creating a Module and attaching a script to it here.