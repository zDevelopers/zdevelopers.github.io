+++
title = "GUI"
description = "Graphical User Interfaces"
+++

This component displays complex Minecraft GUIs with ease.

{{ component(doc="https://dev.zcraft.fr/docs/quartzlib/index.html?fr/zcraft/quartzlib/components/gui/package-summary.html", loader="<code>fr.zcraft.quartzlib.components.gui.Gui</code>") }}

QuartzLib comes with a full-featured component to create GUIs (_Graphical User Interfaces_), either read-only or read-and-write. There is also a specific class to ask for any text through a sign.


# Basic chest GUIs (« action GUI »)

All chest GUIs are created using a class extending [`ActionGui`](https://dev.zcraft.fr/docs/quartzlib/index.html?fr/zcraft/quartzlib/components/gui/ActionGui.html). The only method you have to override is `onUpdate`, called when the GUI is created or updated.

In this method, you'll first have to define the GUI's properties, using the `setSize` and `setTitle` methods. Then, to populate the GUI with items, use the `action` methods. They all take an action name (keep that in mind, we will talk about them later), then a slot ID (from 0 to the size minus one), and finally some kind of ItemStack (an `ItemStack`, an [`ItemStackBuilder`](https://dev.zcraft.fr/docs/quartzlib/index.html?fr/zcraft/quartzlib/tools/items/ItemStackBuilder.html), or some variants with a `Material`, title, lore... directly).

Simple enough. As example, you may have this now:

```java
public class MyGUI extends ActionGui
{
    @Override
    public void onUpdate()
    {
        setTitle(ChatColor.BLACK + "My toasting GUI!");
        setSize(9); // One line

        action("my_action", 0, new ItemStack(Material.CARROT));
        action("close", 8, Material.BARRIER, "Close GUI");
    }
}
```

You can open this GUI using this method—it's the same for **all** QuartzLib GUIs, where `player` is a `Player`.

```java
Gui.open(player, new MyGUI());
```

Ok so, now, if you execute this, you'll have a nice GUI. How to do something when the player clicks somewhere? It's really easy.

Remember the names we gave to the actions? These will be used to reference methods executed when the item representing the action is clicked. You'll just have to annotate the method with `@GuiAction`, like so:

```java
@GuiAction("my_action")
private void the_method_executing_my_action()
{
    // do something (anything!)
}
```

Yes, that's all! To give another example, here is a working implementation of the second action defined in the example:

```java
@GuiAction("close")
private void close_gui()
{
    close();
}
```
(see the javadoc to get a list of usable methods in GUIs; don't forget to check parent classes too!).

If you really _really_ don't like annotations, the component will also look for a method named `action_<action name>`.  
The action methods can either take no argument, or the underlying `InventoryClickEvent`.

Finally, when an action is triggered and no associated method can be found, the following methods are called:
```java
protected void unknown_action(String name, int slot, ItemStack item, InventoryClickEvent event);
protected void unknown_action(String name, int slot, ItemStack item);

// To be exact, the first one calls the second in the default implementation.
```
You can override them to handle such cases.


# GUIs hierarchy

If you use `Gui.open(player, new MyGUI());` and close the GUI, the GUI will be... well, closed. However, if it was opened from
another GUI, you may want to reopen this GUI (think of it like a sub-GUI). If you want this behaviour, open the GUI this way:

```java
Gui.open(player, new MyGUI(), parentGUI);
```

where `parentGUI` is a reference to another GUI, re-opened when the GUI is closed using its method `close()`. As you will typically open sub-GUIs from the parent GUI, the code will look, most of the time, like this:

```java
Gui.open(player, new MyGUI(), this);
```


# GUIs from a collection (« Explorer GUI »)

GUIs are frequently used to display a collection and do actions based on an item of this collection clicked.

Explorer GUIs were created with this use in mind. They can display automatically paginated views of any collection, 1D (list) or 2D (plane).

To get started, create a class extending `ExplorerGui<T>`. The class is parametrized with the data type you want to display — it can be any object, a team, a toast, a player, a block, really anything. In the following example we'll use a Toast (from the zToaster test/example plugin; you can see [the full source code here](https://github.com/zDevelopers/zToaster)).

The method to override is the same: `onUpdate`. You'll still call `setTitle` to change the GUI's title, but leave `setSize` alone: the explorer GUI sets the size itself.

There is a new method for Explorer GUIs: `setMode`, to leave the GUI read&write or read only.

```java
setMode(ExplorerGui.Mode.READONLY);
// or
setMode(ExplorerGui.Mode.CREATIVE);
```

In `CREATIVE` mode, the players will be able to pickup items from the GUI.

To populate the GUI, it is quite different from before. You first have to call `setData` with an array containing the objects to be displayed, ordered.

If you want to display a list, use `setData(T[] data)` directly. For two-dimensional data, add the data width as the second argument. The array will be read like a matrix, with lines stored consecutively; we compute the height automatically.

For advanced use cases, you can directly specify the shape of your data using `setDataShape(int dataWidth, int dataHeight)`.

Now, you must override a method to translate one of your objects into an `ItemStack` to be displayed in the GUI. With our toasts, the method will look like this:

```java
@Override
protected ItemStack getViewItem(Toast toast)
{
    // We here use a simple logic to display a cooked or raw pork based on a
    // “cooked” toast property.
    // But you can do anything you need here, as long as you return an ItemStack.
    if (toast.getStatus().equals(Toast.CookingStatus.COOKED))
    {
        return new ItemStackBuilder(Material.GRILLED_PORK)
                .title("Cooked toast #" + toast.getToastId())
                .item();
    }
    else
    {
        return new ItemStackBuilder(Material.PORK)
                .title("Raw toast #" + toast.getToastId())
                .item();
    }
}
```

To handle the case where the collection is empty, override the `getEmptyViewItem()` method (and return an ItemStack, displayed at the center of the GUI).


## Actions in explorers

Well, you now have a paginated GUI with your data in it. How to execute something when the player clicks an item?

First thing first, **you can define the same actions as in `ActionGui`-based GUIs**. There is a space, between the pagination items links (items to be clicked to go to the next or previous page), where you can put static items with associated actions, just like before. For these, you should use `getSize()` to place them, plus add a specific method: see the example below.

```java
@Override
public void onUpdate()
{
    setData(data);

    // Let's assume we want to add an action at the center of the bottom space
    // in an explorer GUI. As getSize() returns the size of the GUI, we can
    // substract slots from that to place our action at the end of the GUI.
    action("close", getSize() - 4, Material.BARRIER, "Close GUI");

    // ...BUT, with nothing else, the GUI will try to take all the place it can
    // get, and use the last line to put items if all of them can fit in one
    // page. There is an option to force the explorer GUI to keep this line
    // empty no matter what, placing items normally there in another page.
    // Here, we need this option because we use the space to place actions.
    setKeepHorizontalScrollingSpace(true);

    // For 2D GUIs, setKeepVerticalScrollingSpace(boolean) is available too.
}
```

You can also — of course — handle clicks on items from the collection. For that you have three kinds of methods, one for left clicks, one for right clicks, and the last for placements.

```java
protected ItemStack getPickedUpItem(int x, int y);
protected ItemStack getPickedUpItem(T data);
```

These two methods are executed when an item is left-clicked. They must return an `ItemStack` to be placed in the player's cursor, or `null` to cancel the event and put nothing. If not overridden, the item generated by `getViewItem` will be used.

```java
protected void onRightClick(int x, int y);
protected void onRightClick(T data);
```

These two are called when an item is right-clicked. If you want to execute the same action on left and right click, override this one to call `getPickedUpItem`.

```java
protected boolean onPutItem(ItemStack item);
```

This one is called when the player tries to place an item into the GUI. It must return either `true`, to accept the placement and consume the item in the player's cursor, or `false` to cancel the placement.

You can find [in the javadoc](https://dev.zcraft.fr/docs/quartzlib/index.html?fr/zcraft/quartzlib/components/gui/ExplorerGui.html) a lot of other methods to navigate into the GUI (`next()`, `previous()`, etc.).


## Full explorer GUI example

Here is a raw example of a GUI displaying all the `Material` in Minecraft; left-click gives one item; right-click gives a stack.

```java
public class MaterialsGUI extends ExplorerGui<Material>
{
    /**
     * We define a title and a close button
     **/
    @Override
    protected void onUpdate()
    {
        setTitle(ChatColor.BLACK + "All the materials!");
        setData(Material.values());

        action("close", getSize() - 4, Material.BARRIER, "Close GUI");
        setKeepHorizontalScrollingSpace(true);
    }

    /**
     * A lore is added to the items because UX!
     **/
    @Override
    protected ItemStack getViewItem(Material material)
    {
        return new ItemStackBuilder(material)
            .loreLine(ChatColor.GRAY, "Left-click to get one")
            .loreLine(ChatColor.GRAY, "Right-click to get many")
            .item();
    }

    /**
     * This method is called when the close button is clicked
     **/
    @GuiAction("close")
    protected void close_gui()
    {
        close();
    }

    /**
     * We override the picked-up item to give something without lore.
     **/
    protected void getPickedUpItem(Material material)
    {
        return new ItemStack(material);
    }

    /**
     * On right-click, we give an item with the amount set to the
     * max stack size, if available.
     **/
    protected void onRightClick(Material material)
    {
        final ItemStack item = new ItemStack(material);

        if (item.getMaxStackSize() > 0)
            item.setAmount(item.getMaxStackSize());

        return item;
    }
}
```
It's now ready to be opened.

```java
Gui.open(player, new MaterialsGUI());
```


# Updating GUIs

From the inside, a GUI can be updated using it's `update()` method.

There is also a method to update all opened GUIs of a given type. As example, if you have a GUI listing teams — let's call it `TeamsSelectorGUI` —, and you want to update all GUIs in real time when a team is created or removed, call the following method when the teams list is updated:

```java
Gui.update(TeamsSelectorGUI.class);
```

Yes, it's that simple: all opened `TeamsSelectorGUI`-based GUIs will be updated without any user interaction.


# Prompt GUIs

Here is [another kind of GUI](https://dev.zcraft.fr/docs/quartzlib/index.html?fr/zcraft/quartzlib/components/gui/PromptGui.html), used a bit differently. This one will open a sign the player will be able to fill, and return the text entered.

It is meant to be used with a callback. Here's an example:

```java
// Opens a sign and calls the callback with the text entered when the
// sign is closed
PromptGui.prompt(player, lines -> {
    // Do something
});

// Or with a default text from another GUI (typical use):
PromptGui.prompt(player, lines -> {
    // Do something
}, initialContent, parentGUI);
```

**Notice**  
Due to Minecraft limitations, _real signs are placed in the world_, in an empty spot somewhere far in the sky, in a chunk loaded for the player. The sign is then removed when closed. This should not be a problem in most cases but if it is for you, don't use these prompts.
