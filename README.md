# Complete Inventory Plugin Documentation

Welcome to the Complete Inventory plugin! What is this plugin? Well the shortest explanation would be a classic slot-based inventory system and also
a great foundation for similar systems like Crafting and Shops. Here we will have an in-depth look at all of the code as well as plugin integration
into your own project! 🙂

Without further ado, let us begin:

___

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [C++ Classes](#c-classes)
  - [Inventory Component](#inventory-component)
  - [Item Actor](#item-actor)
  - [Item Struct](#item-struct)
  - [Item Use Definitions](#item-use-definitions)
  - [Helper Function Library](#helper-function-library)
- [Widgets](#widgets)
  - [WGB Inventory](#wgb-inventory)
  - [WGB Inventory Slot](#wgb-inventory-slot)
  - [WGB Player Inventory](#wgb-player-inventory)
  - [WGB Container Inventory](#wgb-container-inventory)
  - [WGB Split Widget](#wgb-split-widget)
  - [WGB Item Tooltip](#wgb-item-tooltip)
  - [WGB Drag And Drop](#wgb-drag-and-drop)
- [Regular Blueprints](#regular-blueprints)
  - [BP Chest](#bp-chest)
  - [BP Item Actor](#bp-item-actor)
- [Plugin Installation and Integration](#plugin-installation-and-integration)
  - [First Step](#first-step)
  - [Integrating the HUD](#integrating-the-hud)
  - [Adding New Items](#adding-new-items)
  - [Implementing Input](#implementing-input)
- [What Can You Add to This?](#what-can-you-add-to-this)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->
___

Now lets take a look at everything inside in a project planning style scheme:

![InventoryPluginLayout](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/b4e379c8-b18b-412d-b999-ebda3523de69)

Here you can see how the entire system would look like if it was laid out on paper. Items themselves are represented as structs (or just plain data)
and everything related to storage, manipulation and UI display is connected to the ${\color{red}main \ inventory \ class}$. The files are split into
three categories: ${\color{orange}Classes}$, ${\color{green}Widgets}$ and ${\color{blue}Regular}$ ${\color{blue}Blueprints}$.
  
* I will also include possible additions that you can make to make the plugin to build upon it

Now lets take a closer look at each individual file that we have here:

# C++ Classes

Classes are thoroughly documented in the actual code but we will still briefly go over them here just in case. In case that you maybe forget what something does,
you can always hover over anything in blueprints and it will show you the brief description of the class, function or variable that you are interested in.

<a name="inventory_component"></a>
## Inventory Component
	
This class is the bread and butter of the plugin. Everything from storing items to updating the UI is basically here. It is important to note that said updates
are **not inside** the Event Tick, but rather set up using **custom delegates** that Unreal provides.

* In short, these delegates are just normal variables inside the class, the thing with them is that you can bind events to their execution which, in our case,
  is very useful for updating the UI to any new changes for this component
	
If you want to test out some new items, you can always edit the **Default Items** variable when you attach this component to a pawn. There you can specify what items you
want that pawn to start with. The selection in that array is an FVector2D, where the first number represents the row to look for in a data table (the actual item) and
the second number is how big will its stack be. Note that you can't add over an items limit, in that case it will just default to the maximum stack.

* Adding new items is just adding a row to the ItemDataTable, just make sure you fill out all the information for that row

Equipment slots are set only for Pawns, in our case just the Player for starters. They are only relevant for stat handling and should be only used for pawns that
have the UI to display them (in our case it is the [WGB Player Inventory](#player_inventory_widget)).
	
Stats are handled in the ${\color{orange}ApplyEquipmentStats()}$ method. This is where you would put your code that assigns stats to a pawn or some component that they have.
If you prefer to do that in blueprints, just replace every call (find and replace option in your IDE) of this method with the ${\color{orange}BlueprintApplyEquipmentStats()}$ 
version and then of course do the implementation of the event.

* We will go in detail about the stats in the [FItemStruct](#item_struct) section
* Adding/Removing stats depends on what equipment items the Player has on him. This is done by the above mentioned method every time an item is added/removed from an equipment slot

The rest of the class are all the needed functions for UI interaction like: moving, swapping, merging and splitting items. I've also included a method for instantly splitting an item
in half. Functions are pretty simple themselves, all contain certain null checks and invalid cases and they also utilize our **OnInventoryUpdated** delegate to communicate with the UI.

<a name="item_actor"></a>
## Item Actor

This is the actor that will represent items in the world. Whenever we drop an item, or want to spawn it for example, this is where we will go to. In here we just want to set all the
necessary variables and preform some checks against invalid cases (e.g. actor spawns but has no item). You can set an item for this actor either on the instance in the world or whenever
you attempt to spawn it via blueprints.

The core logic is not here in C++ but rather in the blueprint of this class, but I will save that for when we come to the [Regular Blueprints](#regular_blueprints) section.

<a name="item_struct"></a>
## Item Struct

Here we describe the underlying struct that we will use to represent items in our game. Any kind of relevant information that items need to have goes here. If you want to add any specific property
to your items that isn't there, simply add your variable to it and re-compile the code. 

Now for the actual stats of the items. How do we add new stats to this? Well the process is fairly simple, we just add them to the struct like so:

![AddingStats](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/a523bb38-33a3-42d4-882a-1d8676c47fb0)

After adding your variable (i.e. float CriticalDamage), we should add this to the InitializeTooltipMap() method from Helper Functions:

![TooltipFunction](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/e3edaf92-1a92-4fe0-b4c1-7fbaac587dc3)

And that is all, we have a new stat!

* The **ItemRowStruct** that you can see in the header is just for easily looking up items in the data table. You can easily edit stuff in blueprints with it for whenever you want to add a specific item to an inventory or for debugging purposes.

<a name="item_uses"></a>
## Item Use Definitions

Do you want a specific item to be 'Usable' by actors? If yes, this is the place where we do it. The way it works is as follows:

1. Item is double-clicked in an inventory slot, signalizing that the user wants to use it

2. We go into this class, inside the Master Function

3. We look for the appropriate method for this item

4. Call the method if we found it, if there is no 'Use' implementation then nothing happens

The methods for the items do not have to be C++ functions, they can also be Blueprint Events. In the case of the latter, make sure you implement said events in the blueprint of this class,
otherwise the item will simply do nothing! The reason for doing it like this is that it makes every item flexible. Maybe you don't want to lose one instance of the item upon usage, maybe some other item
loses 5 per use etc.. All of this is easily addable through this class and does not interfere with anything else.

* I have set it up so that only the inventory of the player character will have this set, every other will not. This is just so the items cannot be used outside of his inventory
* Some items, like equipment for example, can share a method for using them. For example the used equipment will just try to be equipped

If you want to use Blueprint Function exclusively for this, just call the blueprint version of the **Master Function** and connect the logic there.

* I've included a way you can do this using the *Names of the Items to call Functions by Name in Blueprints*, this is a pretty straitforward method of 'using' items

## Helper Function Library

This is a Blueprint Function Library that contains plugin-specific functions that make the job in Blueprints much easier. Most of them are simple utility functions like checking if two items are the same, retrieving Item Structs from their Data Table and so on. The biggest part here is the method for calculating drop spaces for items:

![ItemDropFunc](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/2b95621b-6245-491d-ae79-e9fc11815adb)

There is a lot of math happening there so I am going to try to simplify it as much as I can. In a nutshell, this is what the function does:

1. First we need to grab the extent of the Item Actor that is being dropped - this is relevant because items can be of different mesh sizes and we need to take this into account when deciding the width of the trace

2. Following that, we need to find out where the ground is - this is done with a simple trace from the origin of the actor who threw the item to -9999 on the Z axis

- You may need to change this value to **9999** if your world's Z axis is flipped for some reason

3. The middle point between the ground and the start of that trace + the height is where our main box trace starts

4. We generate a random distance that we want to throw the item to

5. Using that distance and the Random Unit Vector, we can calculate where our box trace ends

6. And finally, if that box traces does not hit anything, that is the location we return. Otherwise, we return the location directly down from where the item spawned

This function is crucial in preventing situations like items dropping through walls or other inaccessible areas. Items themselves will simulate physics and will have gravity enabled, so dropping something of a ledge is not a problem. The other issue we also avoid is if we were checking just the final location, we couldn't be certain that the entire path is clear and this can lead to massive problems.

${\color{green}Useful \ Addition:}$

* The 'not found location' scenario depends a lot on what kind of game you have, and you could tweak this to better suit your situation

The other very important function is the ${\color{orange}InitializeTooltipMap()}$ one. This is where we explain how each stat should be described in the tooltip. If you decide to add a new stat to your items,
you should go in here as well to add the appropriate string that will be used to display said stat in the tooltips.

* This is called by the **WGB_Tooltip** every time a new item needs to be initialized for it to display.

# Widgets

<a name="inventory_widget"></a>
## WGB Inventory

Here is the big widget where all the magic happens. This is just the inventory grid itself, the other widgets that are unique to something (Player/Containers) have their own widget but both contain this one.
In this widget we have all the event binding that we need (remember those C++ delegates?), as well as all logic for user interaction. Each widget has just one [Tooltip Widget](#tooltip), which is used across all slots and one [Split Stack Widget](#split_widget), that is used for performing the splitting interaction.

This widget is responsible for creating all the slots for itself. All that is based on the underlying inventory component that is connected to this widget (each inventory can be of different sizes and different
column count). The main part about it is of course handling the *OnMouseDropEvent* that we have whenever an item is dropped here. Lets take a closer look a that function:

![OnMouseDrop_1](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/855d7cbf-13bb-49fd-a93f-ef2bf9d2fca6)

In this first part we need to do all the required initialization. Keep note of the **WhereItemCameFrom** variable, this is needed to enable the cross-inventory interaction and it is set by the
[WGB Inventory Slot](#inventory_slot) upon detecting a drag. We are also using a custom blueprint for the drag and drop so we need to perform a cast to access all of its variables.

After this we simply pass all the required information to the **OnItemDropped** C++ method from the [Inventory Component](#inventory_component) Class:

![OnMouseDrop_2](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/8366b6db-b436-447e-8bd1-a7fa671fb3d5)

From there the function will perform the appropriate action depending on some checks (did we want to merge, split, move etc..).

Splitting is handled by the [WGB Split Widget](#split_widget) and it begins by calling the **OnSplittingInitialized** delegate. This
delegate is hooked up in this widget blueprint as follows:

![Splitting](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/aeacd6e5-f75b-45d1-bc9d-82e041c95c21)

As you may be wondering, well where is the dropping function for items? It exists here in the blueprint, but it does not use it. Well we do use that event but only in the HUD widget. Whenever we drop an item 
outside this widget, it will not get the *OnMouseDrop* event. Instead, the event will be fired in the underlying widget, which in our case is the HUD. That HUD will then handle the event and call the dropping
function and this will conclude our drag and drop operation.

* The HUD basically just handles the drop and is there to contain the WGB Player Inventory and the WGB Container Inventory. In your case you should just add these to **your own HUD**, including the OnMouseDrop event

<a name="inventory_slot"></a>
## WGB Inventory Slot

Widget representing an individual slot inside the inventory. The [Inventory Widget](#inventory_widget) is the one that creates this one and assigns its position in the grid. Here however, we handle stuff 
like detecting a drag, using an item and implementing shotcuts for interaction. Code for the halving of the stack, transfering between inventories and dropping items with a single click is all in here. The dragging and using is connected to the mouse, while halving is connected to a variable in the Player Character. Every slot knows which 
inventory it belongs to and uses the [Tooltip Widget](#tooltip) variable of the WGB_Inventory to display the tooltip upon hovering with the mouse.

How does dragging work? Well it works as foolows:

1. Widget will fire the OnDragDetected event whenever the user starts dragging it with a mouse

2. This will in turn cause an instance of BP_DragAndDrop to be created

3. Using that, we set all the appropriate variables that it will need as well as set its visual representation

4. Releasing the drag will cause the underlying widget (the one that is right below the drop location) to recieve a *OnMouseDrop* event

![SlotDragging](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/1743e888-1000-46d3-bd50-aab9e6613f52)

Each slot can also be assigned to **accept only items of a certain category**. This is set only for equipment slots in the [WGB_Player_Inventory](#player_inventory_widget), but can be used for other stuff as well.
This determines what category of item can be in this slot, anything else will be rejected. If set to 'None', it means that the slot accepts every item regardless of its category.

<a name="player_inventory_widget"></a>
## WGB Player Inventory

Here we have the widget that we put onto our HUD. It contains the regular [WGB Inventory](#inventory_widget) but also some equipment slots for the Player. We initialize 
the equipment slots, which means adding them to the underlying inventory widget's array of slots and giving them the appropriate index. Only other function is to create a Open/Close method for this widget
which is bound to the **OnInventoryOpened** delegate for the underlying inventory component that this widget represents:

![PlayerInventoryOpen](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/804647aa-0694-4ecb-a666-2b51b9a7dd2e)

This is what gets called whenever the appropriate Player Input is triggered.

<a name="container_widget"></a>
## WGB Container Inventory

Represents the inventory of [Chests](#chest) in the project, but can be used for other things like lockers, trash cans etc.. I have included buttons for taking items and transfering gold for easier use, but you can
also transfer items with dragging like usual, the system supports cross inventory interaction.

* This is why we need the **WhereItemCameFrom** variable from our [BP_DragAndDrop](#drag_and_drop), when transfering from the container to the main inventory we need to know which inventory the item came from

Chests interact with this widget by using the **BPI_WidgetInteraction** interface where they call these two methods:

![ContainerInterface](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/e6c3b2c3-f162-4501-88ba-70b75c15b15f)

<a name="split_widget"></a>
## WGB Split Widget

Responsible for performing splits for the inventory widget. Every inventory has just one instance of this widget that is constructed once and just updated when necessary. We use the slider to adjust the amount we want
to split and buttons to confirm or cancel our decision. The text displayed here is how much will be in each stack if you decide to split and that text is of course updated every time the slider value changes. Sliding is set up to be snapping rather than gliding over, the amount of snapping determined of course by how much we can move the slider. The maximal amount of movement depends on several factors, like are we splitting into an empty slot or, if we are not, what is the stack of the other item? This is to prevent overflowing other items by adding more to them than their allowed stack.

* Supports splitting into an empty slot and also into a non-empty slot that has the same item
* Disables the inventory grid while visible, there is no need to execute unnecessary code while this is active

<a name="tooltip"></a>
## WGB Item Tooltip

Here we have our tooltip that shows up when we hover over the slots. Its purely cosmetical and its only purpose is to display appropriate item information to the user. Each inventory has one instance of this widgets and 
the slots just update it and make it visible/hidden when necessary. Customize this in the UMG editor to your liking, I've included a basic design but you should tailor it to your needs.

* The tooltip uses a method from the **Helper Functions** class to get the appropriate infromation for display, this includes a map where the first part is the stat name and the second its value
  
This tooltip widget is only created once per inventory and just updated depending on which slot we are hovering over. Slots are responsibe for calling methods in this widget in order to update its display.

<a name="drag_and_drop"></a>
## WGB Drag And Drop

Created by the [Inventory Slot](#inventory_slot) whenever the user starts dragging. The only thing we have here are actually the variables that we need to 'carry' in the drag. WGB Drag Visual is just as its name implies, the visual representation of the item being dragged. If you wanted to, for example, change how the dragging looks, you would just need to edit the WGB Drag Visual and that would be all.

* This is automatically destroyed when the dragging operation ends (some widget handles the OnMouseDrop event)

<a name="regular_blueprints"></a>
# Regular Blueprints

<a name="chest"></a>
## BP Chest

As the name implies, this is just a basic blueprint of a chest. Its purpose is to store items and interact with the UI of the Player. Each chest's inventory can of course be customized in both size and what it holds. You can set this gloabally for all chests but also edit it per instance in the world. Chest interaction is just a simple overlap check with the box that is in front
of it. This can only happen with the Player character. Upon overlapping, the following happens:

1. We first need to pass the chest's inventory to the WGB Container Inventory from the Player's HUD

2. After that we initialize it

3. Lastly we just need to make it visible and also open the inventory of the Player if it wasn't opened already

From this point all the interaction is UI based, however I also included the option that the chests just drop all items upon overlapping. This, alongside what is in the chest's inventory, can be set on each instance
of it in the world, or just defaulted across all of them using the DefaultItems array from the Inventory Class:

![ChestComp](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/841a2992-a54b-4f5d-aaac-8e553d890c5b)

<a name="item_actor"></a>
## BP Item Actor

Adding an item to an inventory happens when the Player overlaps with this actor. In the case of everything being added (nothing was left over) the actor is destroyed, otherwise it still remains in the world. The biggest thing that we have here is the Drop Effect. The drop effect happens using a spline component. This component will have three points that the item will follow:

1. The starting location of this actor (wherever it was spawned), this is set to the origin of the actor who 'threw' the item away

2. The final location which is generated by the ${\color{orange}Find \ Item \ Drop \ Location}$ method inside the [BFL functions](#bfl) blueprint

3. Middle point between the first two locations

This drop effect only happens on items that are spawned via code, the ones placed in the world will not have this. It is important to note that the collision on this actor is set to identify it as a **World Dynamic** object. You can edit the time required for the drop in a simple variable in this blueprint, and also the maximum height it can reach.

* This height is basically just a value used to calculate the highest point the item will reach when performing a drop. This is relevant so that we can determine the size of the trace for it in the ${\color{orange}Find \ Drop \ Location}$ method from the Helper Functions class.

${\color{green}Useful \ Addition:}$

* You may not want the **WorldDynamic** type and prefer to use a custom collision channel, which you of course can. Just don't forget to set up that channel correctly.

The actor has a default mesh assigned to it that will be used in case the item it is representing does not have a specific mesh connected to it. I highly recommend setting this to something:

![ItemActorMesh](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/23c57875-8a49-4ba0-9655-880e368eec6b)

<a name="integration"></a>
# Plugin Installation and Integration

First and foremost, lets get the plugin files to your project. Navigate to your Unreal Engine install path and go into the Engine/Plugins folder for your engine version. There you will find my plugin (folder name is
'CompleteInventory'). To keep it very clean and short, here is what you need to do step by step:

1. In the Epic Games launcher, go to the 'Library' section of the Unreal Engine page and find the 'Complete Inventory' Plugin

2. Click 'Install to Engine' and then click the version that you are using

3. Create a 'Plugins' folder in your own project (capitalization is important) if you do not have one already

4. Go to where you installed Unreal Engine, then into the version of it (e.g C:\Unreal Engine\UE 5.1)

5. Copy over the folder called 'Complete Inventory' from the Engine/Plugins/Marketplace folder into your 'Plugins' folder

- Optional but recommended: Remove that plugin folder from the engine path but save it somewhere else in case you need the default files 

6. Load your project and go to the 'Edit' Tab

7. Click Plugins and search for 'CompleteInventory'

8. Enable the Plugin, restart the editor and compile everything

Okay so with that we have gotten this plugin into your project, what now? Well now it is just a matter of replacing everything that is 'default' to the plugin with your own actors and UI.

* You will be modifying the files of the plugin but since you are doing it in the 'Plugins' folder, the original files
in the engine will remain the same. If something goes wrong just delete the plugins folder and repeat the above steps,
you will get fresh files and there should not be any problem

* If you have any issues with the compile, close the editor and delete the **'Binaries' and 'Intermediates'** folder in the plugin directory (Your Project/Plugins/CompleteInventory) and then re-launch

## First Step

Lets begin by adding an Inventory Component to your Player or any other Actor that you wish to use it on:

![AddingInventoryComp](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/f5588567-3447-4984-bfdf-10a729c40935)

You can then customize the component by selecting it and going over to the details panel:

![InventoryCustomization](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/7e639364-a2af-4805-b1ae-c5f73705669d)

## Integrating the HUD

What you want to do with you HUD is pretty simple, just add the WGB Player Inventory widget and the WGB Container Inventory widget to it and implement the **OnMouseDrop** event for it like so:

![HUD_Drop](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/eac85800-1ae0-433f-8572-a934927745cc)

You can literally copy over the same thing we have in the WGB_HUD. If you do not want to drop items by dragging them onto the HUD, you can skip this part completely!

## Adding New Items

Adding Items is as simple as creating a new row in the Item Data Table like so:

![ItemAdding](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/e91d306f-d15a-41a8-9212-b667e8f495ea)

When you add a new row, fill out the information below and you have your first custom item!

## Implementing Input

As for the input it depends whether you are using *Enhanced Input* or not. My plugin goes along the Enhanced Input path but you can easily also not use it. If that is the case,
you will need to replace the EI events with a 'Key Pressed' event or your own **Action Mapping**. To connect the new inputs simply copy over the following section from the **BP_CompleteInventoryCharacter**:

![Input](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/49f8252b-e999-449d-af57-2f4ec3ebeb9c)

This concludes the integration and you should be able to play around with it 🙂

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

You can also add item drops to your enemies, just add an inventory component to them, generate their items from a data table and integrate something similar to the chest's DropAllItems event.

___

That will be all for now, I know this was a hefty read but I hope you learned something and can start building upon all of this. Thank you for getting my plugin and I hope it serves you well 😉
