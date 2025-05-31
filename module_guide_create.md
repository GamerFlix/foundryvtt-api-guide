# Setting up a module
This guide will take you through the steps of creating a module and adding a script to it. It will however not go over writing the relevant code which will be the topic of a future guide. If you want to follow along but don't have a script to attach you can [refer to this guide](https://github.com/GamerFlix/foundryvtt-api-guide/blob/main/maximum-zoom-module.md) which goes over the nessecary code to adjust the maximum zoom level of the canvas.

## Creating the module 
In order to have a module to attach scripts to you first need to create it. Should you already have one for sharing Compendiums between worlds you can use that one though it doesn't hurt to make a second one to be able to deactivate all scripts without removing your Compendiums. Luckily Foundry takes care of the creation for you with an easy to use GUI. To access it go to the setup page. and click the little cog Icon next to "Update All" on the Add-on Modules tab:

![create_module](https://github.com/user-attachments/assets/c4e5da14-225b-4d2d-b220-dcb43468c322)

On the next page you can fill in various things like the module title that is later displayed in your manage modules list. You don't need to fill in everything and the default values are largely fine as you can edit them later if you wish to do so. There is however **one exception** to this: The identifier of the Package. **The identifier can not be changed later** (without breaking everything depending on it) so try to think of one that is unlikely to collide with anything else.

![Id_is_important](https://github.com/user-attachments/assets/3549317c-4c78-432e-9e77-c35c8fc35c49)

Once you are happy with your choices hit create Module and the Module will appear in your list. Now that you have your own Module shutdown Foundry and proceed to the next step.

## Adding scripts to a package
To attach a script to a Module it is crucical that you first **shutdown Foundry** if you haven't done so yet. This is of vital importance as otherwise any changes you attempt to make to your Module's Manifest will not save.
After you shutdown Foundry navigate to your userdata Folder. The *default* locations depend on your OS. These are:
Windows 10/11 - %localappdata%\FoundryVTT\
macOS - ~/Library/Application Support/FoundryVTT
Linux - ~/.local/share/FoundryVTT

Once you have found your userdata navigate to the `modules` folder and then the folder of your module, the name of which is identical to the identifier you chose earlier:

![find module](https://github.com/user-attachments/assets/7159a36b-e740-4426-b19e-4899473c48c9)

Open the folder, create a new text file and give it a name of your choice. Change the extension to `.js`. If you are on Windows you may need to enable showing those first a tutorial to which can be found [here](https://support.microsoft.com/en-us/windows/common-file-name-extensions-in-windows-da4a4430-8e76-89c5-59f7-1cdbbc75cb01).

![grafik](https://github.com/user-attachments/assets/3c61ddbe-c16d-4428-b185-642ee881f69a)

Add your code to the `.js` file and save it. Now go back to the folder of your module and edit the Manifest. Since this is a Module it'll be called `module.json`. Once opened it should look something like this though it may have more fields if you also have Compendium Packs or dependencies specified.

![grafik](https://github.com/user-attachments/assets/87180019-6f41-406b-8cd0-f62e614cd303)

Add a new line **in the root of the object** so directly inside the brackets on the first and last line **not** inside another set of brackets therein. Add a key of `"esmodules"` with a value of an Array in there and keep an eye on the commas. They should be between each element but not after the last on of each Object/Array. An example can be seen here:

![grafik](https://github.com/user-attachments/assets/3338bd58-1aec-4f8d-bbf4-6783f9024146)

Add the name of your file inside these brackets. In my case I called the file `myScript.js` so that is what I will add to the Array **within quotation marks**:

![grafik](https://github.com/user-attachments/assets/51a4e0f2-f9ea-4627-be15-db310de690f0)

**Save the file then close it.** Open Foundry back up and check if the module still appears in your list. If not you likely have an errant comma or similar in your Manifest. In this case I recommend using an IDE such as [VSCode](https://code.visualstudio.com/) or an [online JSON linter](https://jsonlint.com/) to find the issue, remember to edit it only while Foundry is shut down. You can also always ask for additional assistance on the [official FoundryVTT Discord server](https://discord.com/invite/foundryvtt)

If everything appears fine and you haven't added your code to the `.js` file yet do so now and save it. In order for this to take effect you will need to refresh though, for edits of `.js` files you do not need to fully restart as that is only nessecary when editing the manifest.

Now you have your very own Module complete with a script.
