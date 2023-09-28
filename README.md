# Complete Inventory Plugin Documentation

Welcome to the Complete Inventory plugin documentation! Here we will have a look at how you should integrate this plugin in your project and
I will show you how to do some common stuff within the plugin! 🙂

Without further ado, let us begin:
___

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
- [Plugin Installation and Integration](#plugin-installation-and-integration)

- [Plugin Installation and Integration](#plugin-installation-and-integration)
  - [First Step](#first-step)
  - [Integrating the HUD](#integrating-the-hud)
  - [Adding New Items](#adding-new-items)
  - [Adding New Stats](#adding-new-stats)
  - [Implementing Input](#implementing-input)
  - [Adding the Plugin Module](#adding-the-plugin-module)
- [What Can You Add to This?](#what-can-you-add-to-this)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->
___


<a name="integration"></a>
# Plugin Installation and Integration

First and foremost, lets get the plugin files to your project. Navigate to your Unreal Engine install path and go into the Engine/Plugins folder for your engine version. There you will find my plugin (folder name is
'CompleteInventory'). To keep it very clean and short, here is what you need to do step by step:

1. In the Epic Games launcher, go to the 'Library' section of the Unreal Engine page and find the 'Complete Inventory' Plugin

2. Click 'Install to Engine' and then click the version that you are using

3. Create a 'Plugins' folder in your own project (capitalization is important) if you do not have one already

4. Go to where you installed Unreal Engine, then into the version of it (e.g C:\Unreal Engine\UE 5.1)

5. **Copy over the folder called 'Complete Inventory' from the Engine/Plugins/Marketplace folder into your project's 'Plugins' folder** 

	* You will be modifying the files of the plugin but since you are doing it in the 'Plugins' folder, the original files
	in the engine will remain the same. If something goes wrong just delete the plugins folder and repeat the above steps,
	you will get fresh files and there should not be any problem

6. Load your project and go to the 'Edit' Tab

7. Click Plugins and search for 'CompleteInventory'

8. Enable the Plugin, restart the editor and compile everything

	* If you have any issues with the compile, close the editor and delete the **'Binaries' and 'Intermediates'** folder in the plugin directory (Your Project/Plugins/CompleteInventory) and then re-launch
   
Okay so with that we have gotten this plugin into your project, what now? Well now it is just a matter of adding the necessary components to your
specific actors and adding a couple of widgets to your HUD and you are good to go 🙂

## First Step

Lets begin by adding an Inventory Component to your Player or any other Actor that you wish to use it on:

![AddingInventoryComp](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/f5588567-3447-4984-bfdf-10a729c40935)

You can then customize the component by selecting it and going over to the details panel:

![InventoryCustomization](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/7e639364-a2af-4805-b1ae-c5f73705669d)

## Integrating the HUD

What you want to do with you HUD is pretty simple, just **add the WGB Player Inventory widget and the WGB Container Inventory widget** to it and implement the **OnMouseDrop** event inside you HUD like so:

![HUD_Drop](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/e05e431f-5b2d-4cc6-b965-0d8ad14256a2)

If you do not want to drop items by dragging them onto the HUD, you can skip this part completely!

## Adding New Items

Adding Items is as simple as creating a new row in the Item Data Table like so:

![ItemAdding](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/e91d306f-d15a-41a8-9212-b667e8f495ea)

When you add a new row, fill out the information below and you have your first custom item!

## Adding New Stats

For stats, there are just two things you need to do, first is to declare the stat that you desire inside the **FItemStruct** and
the second is to add the appropriate map entry in the following function:

![TooltipMap](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/cb8292c2-e8e2-46f8-a2a3-fd8eac7ec8f6)

This is how you stat will be displayed in the item tooltip widget.

## Implementing Input

As for the input it depends whether you are using *Enhanced Input* or not. My plugin goes along the Enhanced Input path but you can easily also not use it. If that is the case, you will need to replace the EI events with a 'Key Pressed' event or your own **Action Mapping**. To connect the new inputs you should first **add new inputs in your own IMC_MappingContext** and then simply copy over the following section from the **BP_CompleteInventoryCharacter**:

![Input](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/74b29cdf-15f5-4ae1-862f-ec9b81878952)

This concludes the integration and you can start playing around with everything 🙂

## Adding the Plugin Module

Before trying to work with files from the plugin, you should include the plugin module to your project's **Build.cs** file like so:

![BuildCS_File](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/2eac1709-a3ca-4ea6-959f-c0cd254eccb8)

After doing this you can include headers, declare variables and call methods from the plugin without any issues. If you get linking
errors when trying to do the above things, that means that you forgot to add the plugin module to your build.cs file.

# What Can You Add to This?

Here I am going to mention some things that come to my mind when thinking about expanding on this plugin:

1. Shop system

Items already have a value assigned to them, adding a shop would be pretty straightforward. You would need a shop widget and to make transfers between the
shop and the player only happen if the player has enough currency to buy the item. Take into account item selling and that is pretty much it!

2. Item Rarity

A simple enum in FItemStruct will do the trick, declare it like the EItemCategory enum is and add all the rarities you want. For what the rarities do I cannot tell
you as it is very dependant on your project and what you want to accomplish with the rarities. Just make whatever actor/UI has items that it uses a 'GenerateRarity'
method in its BeginPlay event. This generation again depends on how you want to do it.

3. Loot System

As the items are just data basically, you can easily set up this system so that chests, other containers and items in the world are randomly generated upon game start.
This again depends on how you want to do it for your project. One thing I can suggest if you are leaning more into C++ is to create an interface for the system and just
add it to every actor that should generate items.

___

That will be all for now, I hope you the plugin will serve you well. For any support issues be sure to come over to my [discord](https://discord.gg/yrFwH5rMrN) channel
where you can ask me anything you want about the plugin. Thank you for getting my plugin, have a nice day 😉
