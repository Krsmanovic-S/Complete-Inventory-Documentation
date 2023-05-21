# Complete Inventory Plugin Documentation

Welcome to the Complete Inventory plugin! What is this plugin? Well the shortest explanation would be a classic RPG-style inventory system and also
a great foundation for similar systems like Crafting and Shops. Here we will have an in-depth look at all of the code as well as plugin integration
into your own project. In this first section, I will first list all the features and following that I'll show you the general principles of this 
plugin and how all the classes and blueprints interact with each other.

Without further ado, lets begin with all the features as well as what this plugin can offer you:
* Slot based system
* Data table driven item creation
* Moving/Swapping/Merging/Splitting items
* Ability to stack items and configure the maximum stack
* Interaction between multiple inventories
* Picking up items from the ground
* Having currency inside the inventory
* Dropping items from the inventory
* Nothing in Event Tick, blueprint casting reduced to a minimum
* Clean code that is easily expandable

___

# TOC
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

Here we describe the underlying struct that we will use to represent items in our game. Any kind of relevant information that items need to have goes here.

${\color{green}Useful \ Addition:}$
	
* If you want your item actors to look like the item that they are carrying, you can add a *UStaticMeshComponent* here and then use that in the [Item Actor](#item_actor) class to set its mesh in BeginPlay

Now for the actual stats of the items. How do we add new stats to this? Well the process is as follows:

1. Create a variable in the struct to represent the stat (e.g. float CriticalDamage)
2. Handle the addition/removal in the ${\color{orange}ApplyEquipmentStats()}$ method in the inventory (e.g. Player->pCriticalDamage += Item.CriticalDamage)
3. Add the appropriate text in the ${\color{orange}InitializeTooltipMap()}$ method in the [Inventory Class](#inventory_component) (e.g. {" Critical Damage", TooltipItem.CriticalDamage},
4. Use the stat when doing other calculations (depends on your project of course)

<a name="item_uses"></a>
## Item Use Definitions

Do you want a specific item to be 'Usable' by actors? If yes, this is the place where we do it. The way it works is as follows:

1. Item is double-clicked in an inventory slot, signalizing that the user wants to use it
2. We go into this class, inside the Master Function
3. We look for the appropriate method for this item
4. Call the method

The methods for the items do not have to be C++ functions, they can also be Blueprint Events. In the case of the latter, make sure you implement said events in the blueprint of this class,
otherwise the item will simply do nothing! The reason for doing it like this is that it makes every item flexible. Maybe you don't want to lose one instance of the item upon usage, maybe some other item
loses 5 per use etc.. All of this is easily addable through this class and does not interfere with anything else.

* I have set it up so that only the inventory of the player character will have this set, every other will not. This is just so the items cannot be used outside of his inventory
* Some items, like equipment for example, can share a method for using them. For example the used equipment will just try to be equipped

# Widgets

<a name="inventory_widget"></a>
## WGB Inventory

Here is the big widget where all the magic happens. This is just the inventory grid itself, the other widgets that are unique to something (Player/Containers) have their own widget but both contain this one.
In this widget we have all the event binding that we need (remember those C++ delegates?), as well as all logic for user interaction. Each widget has just one [Tooltip Widget](#tooltip), which is used across all slots and one [Split Stack Widget](#split_widget), that is used for performing the splitting interaction.

This widget is responsible for creating all the slots for itself. All that is based on the underlying inventory component that is connected to this widget (each inventory can be of different sizes and different
column count). The main part about it is of course handling the *OnMouseDropEvent* that we have whenever an item is dropped here. Lets take a closer look a that function:

![Capture1](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/7d136d02-78bf-480d-8186-0ed86f5925f5)

In this first part we need to do all the required initialization. Keep note of the **WhereItemCameFrom** variable, this is needed to enable the cross-inventory interaction and it is set by the
[WGB Inventory Slot](#inventory_slot) upon detecting a drag. We are also using a custom blueprint for the drag and drop so we need to perform a cast to access all of its variables.

After this we need to determine where did we drop the item. Is it even on any slot? If it is, does that slot have an item? Are we trying to split the item? These are the questions we need to answer until we decide
what to do with the item. Finally, after resolving all of that, this is where we handle the appropriate interaction for the drop:

![Capture3](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/66f54ceb-57ae-4fbe-808b-4c6552208eab)

These functions provide the [Inventory Component](#inventory_component) class with the appropriate information and call its respective method for what needs to be done (MoveItemToEmptySlot, SwapItems etc..).
Splitting is handled by the [WGB Split Widget](#split_widget).

* Note that for the splits, the widget has to access certain variables in the Player Character blueprint, you will have to add these variables to your own character to make it work but don't worry,
we go in detail about this in the [Integration](#integration) section

As you may be wondering, well where is the dropping function for items? It exists here in the blueprint, but it does not use it. Well we do use that event but only in the HUD widget. Whenever we drop an item 
outside this widget, it will not get the *OnMouseDrop* event. Instead, the event will be fired in the underlying widget, which in our case is the HUD. That HUD will then handle the event and call the dropping
function and this will conclude our drag and drop operation.

* The HUD basically just handles the drop and is there to contain the WGB Player Inventory and the WGB Container Inventory. In your case you should just add these to **your own HUD**, including the OnMouseDrop event

<a name="inventory_slot"></a>
## WGB Inventory Slot

Widget representing an individual slot inside the inventory. The [Inventory Widget](#inventory_widget) is the one that creates this one and assigns its position in the grid. Here however, we handle stuff 
like detecting a drag, using an item and halving a stack. The dragging and using is connected to the mouse, while halving is connected to a variable in the Player Character. Every slot knows which 
inventory it belongs to and uses the [Tooltip Widget](#tooltip) of the inventory to display the tooltip upon hovering with the mouse.

How does dragging work? Well it works as foolows:

1. Widget will fire the OnDragDetected event whenever the user starts dragging it with a mouse
2. This will in turn cause an instance of BP_DragAndDrop to be created
3. Using that, we set all the appropriate variables that it will need as well as set its visual representation
4. Releasing the drag will cause the underlying widget (the one that is right below the drop location) to recieve a *OnMouseDrop* event

Each slot can also be assigned to **accept only items of a certain category**. This is set only for equipment slots in the [WGB_Player_Inventory](#player_inventory_widget), but can be used for other stuff as well.
This determines what category of item can be in this slot, anything else will be rejected. If set to 'None', it means that the slot accepts every item regardless of its category.

<a name="player_inventory_widget"></a>
## WGB Player Inventory

Here we have the widget that we put onto our HUD. It contains the regular [WGB Inventory](#inventory_widget) but also some equipment slots for the Player. The only function we have here is to initialize 
the equipment slots, which means adding them to the underlying inventory widget's array of slots and giving them the appropriate index.

<a name="container_widget"></a>
## WGB Container Inventory

Represents the inventory of [Chests](#chest) in the project, but can be used for other things like lockers, trash cans etc.. I have included buttons for taking items and transfering gold for easier use, but you can
also transfer items with dragging like usual, the system supports cross inventory interaction.

* This is why we need the **WhereItemCameFrom** variable from our [BP_DragAndDrop](#drag_and_drop), when transfering from the container to the main inventory we need to know which inventory the item came from

Blueprint for the chest is where this is initialized and used, so look there for more detail on how we get to it showing up on the HUD.

<a name="split_widget"></a>
## WGB Split Widget

Responsible for performing splits for the inventory widget. Every inventory has just one instance of this widget that is constructed once and just updated when necessary. We use the slider to adjust the amount we want
to split and buttons to confirm or cancel our decision. The text displayed here is how much will be in each stack if you decide to split and that text is of course updated every time the slider value changes.

* Supports splitting into an empty slot and also into a non-empty slot that has the same item
* Disables the inventory grid while visible, there is no need to execute unnecessary code while this is active

<a name="tooltip"></a>
## WGB Item Tooltip

Here we have our tooltip that shows up when we hover over the slots. Its purely cosmetical and its only purpose is to display appropriate item information to the user. Each inventory has one instance of this widgets and 
the slots just update it and make it visible/hidden when necessary. Customize this in the UMG editor to your liking, I've included a basic design but you should tailor it to your needs.

* The tooltip uses a map variable from the [Inventory Class](#inventory_component) to represent the stats as text in a certain format. If you want them to display as percentages make sure to add it to the ${\color{orange}InitializeTooltipMap()}$ method in the inventory class
* Stat creation is done by creating WGB Individual Stat widgets for every single stat, this widget is just a simple text block

<a name="drag_and_drop"></a>
## WGB Drag And Drop

Created by the [Inventory Slot](#inventory_slot) whenever the user starts dragging. The only thing we have here are actually the variables that we need to 'carry' in the drag. WGB Drag Visual is just as its name implies, the visual representation of the item being dragged.

* This is automatically destroyed when the dragging operation ends (some widget handles the OnMouseDrop event)

<a name="regular_blueprints"></a>
# Regular Blueprints

<a name="chest"></a>
## BP Chest

As the name implies, this is just a basic blueprint of a chest. Its purpose is to store items and interact with the UI of the Player. Chest interaction is just a simple overlap check with the box that is in front
of it. This can only happen with the Player character. Upon overlapping, the following happens:

1. We first need to pass the chest's inventory to the WGB Container Inventory from the Player's HUD
2. After that we initialize it
3. Lastly we just need to make it visible and also open the inventory of the Player if it wasn't opened already

From this point all the interaction is UI based, however I also included the option that the chests just drop all items upon overlapping. This, alongside what is in the chest's inventory, can be set on each instance
of it in the world, or just defaulted across all of them using the DefaultItems array from the Inventory Class.

<a name="item_actor"></a>
## BP Item Actor

Adding an item to an inventory happens when the Player overlaps with this actor. In the case of everything being added (nothing was left over) the actor is destroyed, otherwise it still remains in the world. The biggest
thing that we have here is the Drop Effect. I'd like to say more about it as it could confuse many people.

The drop effect happens using a spline component. This component will have three points that the item will follow:

1. The starting location of this actor (wherever it was spawned), this is set to the origin of the actor who 'threw' the item away
2. The final location which is generated by the ${\color{orange}Find \ Location \ For \ Item \ Drop}$ method inside the BFL functions blueprint
3. Middle point between the first two locations

This drop effect only happens on items that are spawned via code, the ones placed in the world will not have this. It is important to note that the collision on this actor is set to identify it as a **World Dynamic** object. You can edit the time required for the drop in a simple variable in this blueprint, and also the maximum height it can reach.

* This height is basically just a value used to calculate the highest point the item will reach when performing a drop. This is relevant so that we can determine the size of the trace for it in the ${\color{orange}Find \ Location \ For \ Item \ Drop}$ method

${\color{green}Useful \ Addition:}$

* You may not want the **WorldDynamic** type and prefer to use a custom collision channel, which you of course can. Just don't forget to set up that channel correctly.

<a name="bfl"></a>
## BFL Functions

The only key function here that we will look at closely is the ${\color{orange}Find \ Location \ For \ Item \ Drop}$. There is a lot of math happening there so I am going to try to simplify it as much as I can.
In a nutshell, this is what the function does:

1. First we need to grab the extent of the Item Actor that is being dropped - this is relevant because items can be of different mesh sizes and we need to take this into account when deciding the width of the trace
2. Following that, we need to find out where the ground is - this is done with a simple trace from the origin of the actor who threw the item to -9999 on the Z axis
	* You may need to change this value to 9999 if your world's Z axis is flipped for some reason
3. The middle point between the ground and the start of that trace + the height is where our main box trace starts
4. We generate a random distance that we want to throw the item to
5. Using that distance and the Random Unit Vector, we can calculate where our box trace ends
6. And finally, if that box traces does not hit anything, that is the location we return
7. Otherwise, we return the location directly down from where the item spawned

This function is crucial in preventing situations like items dropping through walls or other inaccessible areas. Items themselves will simulate physics and will have gravity enabled, so dropping something of a ledge
is not a problem. The other issue we also avoid is if we were checking just the final location, we couldn't be certain that the entire path is clear and this can lead to massive problems.

${\color{green}Useful \ Addition:}$

* The 'not found location' scenario depends a lot on what kind of game you have, and you could tweak this to better suit your situation

<a name="integration"></a>
# Plugin Installation and Integration

First and foremost, lets get the plugin files to your project. Navigate to your Unreal Engine install path and go into the Engine/Plugins folder for your engine version. There you will find my plugin (folder name is
'CompleteInventory'). To keep it very clean and short, here is what you need to do step by step:

1. Create a 'Plugins' folder in your own project (capitalization is important)
2. Copy over those files from the Engine/Plugin folder into your 'Plugins' folder
3. Load your project and go to the 'Edit' Tab
4. Click Plugins and search for 'CompleteInventory'
5. Enable the Plugin, restart the editor and compile everything

Okay so with that we have gotten this plugin into your project, what now? Well now it is just a matter of replacing everything that is 'default' to the plugin with your own actors and UI.

* You will be modifying the files of the plugin but since you are doing it in the 'Plugins' folder, the original files
in the engine will remain the same. If something goes wrong just delete the plugins folder and repeat the above steps,
you will get fresh files and there should not be any problem

* If you have any issues, close the editor and delete the **'Binaries' and 'Intermediates'** folder in the plugin directory (Your Project/Plugins/CompleteInventory)

## First Step

Lets begin the integration shall we? This includes stuff like replacing the references from the **BP_CompleteInventoryCharacter** to your own, setting up the HUD,
adding some variables and connecting the input. Before we touch anything, just simply add the Inventory Component to your character like this:

![Capture4](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/96f27f2d-971b-41f4-ace3-5db7dbb5439f)

## Integrating the HUD

What you want to do with you HUD is pretty simple, just add the WGB Player Inventory widget and the WGB Container Inventory widget to it and implement this part in your HUD's BeginPlay:

![Capture2](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/7df36c05-e55c-4cdd-8ba6-3607043d4160)

* Note of course that the **OwningPlayer** variable has to be your own character blueprint, I will list all the spots for this later on

* The Health Bar is of course purely optional, it was just for demonstration purposes for using items

## Adding Character Variables

Go to your character blueprint and add these variables (change the HUD variable to your own HUD or use this one if
you don't have one):

![Capture](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/111c7897-02e6-4872-b460-cb91af108aed)

* Don't forget to **initialize your HUD** in the BeginPlay, for reference take a look at the BeginPlay of the plugin character

## Implementing Input

As for the input it depends whether you are using *Enhanced Input* or not. My plugin goes along the Enhanced Input path but you can easily also not use it. If that is the case,
you will need to replace the EI events with a 'Key Pressed' event or your own **Action Mapping**. To connect the new inputs simply do the following:

1. Open your mapping context (for the plugin it is called IMC_Default, look for something similar)
2. Add **three** new mappings and assign them IA_SplitItem, IA_HalveStack and IA_OpenInventory
3. Assign some different keys if you do not like the preset ones
4. Done!

Now that we have that up its time to copy over the input events. Copy the following into your character blueprint:

![Capture](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/fb3e3a8c-91dc-46f2-894e-4771ab47909a)

![Capture1](https://github.com/Krsmanovic-S/Complete-Inventory-Documentation/assets/103185975/e762a884-743c-420a-a409-20b9ab16f426)

Connect your variables to the input events like in the pictures.

## Changing Character Variable Type

Finally, lets see in which files you will need to change the variable type of the plugin character to your own:

1. Inside the **WGB HUD**, change **OwningPlayer**

2. Inside **WGB Inventory**, change **PlayerReference** as well as the **'Cast To'** node in BeginPlay
 
3. Same as above but in the **WGB Container Inventory** (also in BeginPlay for this widget) 

4. In **BP_ItemUseDefinitions** change **PlayerReference** 

5. Update the **PlayerReference** variable in the **ConstructItemUseBlueprint** method of the WGB Inventory

6. Change the **Player** variable in BP_Chest and update **'Cast To'** node in BeginPlay

* When you change these variables, you will need to reconnect the pins for the nodes where the variabe was used.

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

__

That will be all for now, I know this was a hefty read but I hope you learned something and can start building upon all of this. Thank you for getting my plugin and I hope it serves you well 😉
