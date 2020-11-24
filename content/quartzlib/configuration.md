+++
title = "Configuration | QuartzLib"
description = "Configuration"
+++

This component centralizes the configuration of your application, providing simple access and automatic parsing.

{{ component(doc="https://dev.zcraft.fr/docs/quartzlib/index.html?fr/zcraft/quartzlib/components/configuration/package-summary.html", loader="<em>Your configuration class (see below)</em>") }}

# The concept

The old way to manage configuration is pretty annoying. You have to access the main plugin class everytime, the data types are limited, the configuration paths are hard-coded...

Using QuartzLib, you transform this...

```java
final EntityType entity;
try
{
    entity = EntityType.valueOf(MainPluginClass.getInstance().getConfig().getString("my_section.entity", "ZOGLIN"));
}
catch (IllegalArgumentException e)
{
    entity = EntityType.ZOGLIN;
    MainPluginClass.getInstance().getLogger().warning("Invalid entity set in config; using default value.");
}
```

...into this:

```java
final EntityType entity = Config.MY_SECTION.ENTITY.get();
```

Better, eh?

# Writing the configuration class

Behind the scenes, there is a class storing the configuration scheme.
Here is the only small inconvenient of this configuration management: you have to duplicate the config in both the `config.yml` file and this class. But it's a small task, plus a tool exist to generate the configuration class from the `config.yml` (we'll talk about this later).

You'll have to write a class extending the [`Configuration`](https://zdevelopers.github.io/QuartzLib/index.html?fr/zcraft/zlib/components/configuration/Configuration.html) class. Let's call this class `Config`.

```java
import fr.zcraft.quartzlib.components.configuration.Configuration;

public final class Config extends Configuration
{

}
```

For everything to work, load this class in your main plugin class:

```java
loadComponents(Config.class);
```

## Configuration entries

Then you add the configuration entries. The idea is to add a public and static field per entry, the data type being a [`ConfigurationItem<?>`](https://zdevelopers.github.io/QuartzLib/index.html?fr/zcraft/zlib/components/configuration/ConfigurationItem.html).

```java
import fr.zcraft.quartzlib.components.configuration.Configuration;
import fr.zcraft.quartzlib.components.configuration.ConfigurationItem;

// I highly recommend to import this statically
import static fr.zcraft.quartzlib.components.configuration.ConfigurationItem.item;

public final class Config extends Configuration
{
    public static ConfigurationItem<String> MY_CONFIG = item("my_config", "default value");
}
```

Here I created a `String` configuration item, related to the key `my_config` in the `config.yml` file. Using it like this will return the value as a `String` from the configuration file, or `"default value"` if the key is missing.

```java
final String value = Config.MY_CONFIG.get();
```

## Referencing sub-keys

Let's say you have this configuration file:

```yaml
title: My title
display-as:
    scoreboard: true
    action-bar: true
```

You can write the `Config` class in two different ways. The most intuitive one is to reference the sub-keys with the dot notation, just like Bukkit:

```java
public final class Config extends Configuration
{
    public static ConfigurationItem<String> TITLE = item("title", "My title");
    public static ConfigurationItem<Boolean> DISPLAY_AS_SCOREBOARD = item("display-as.scoreboard", true);
    public static ConfigurationItem<Boolean> DISPLAY_AS_ACTION_BAR = item("display-as.action-bar", true);
}
```

But you can also use sub-classes, like this:

```java
import fr.zcraft.quartzlib.components.configuration.Configuration;
import fr.zcraft.quartzlib.components.configuration.ConfigurationItem;
import fr.zcraft.quartzlib.components.configuration.ConfigurationSection;

import static fr.zcraft.quartzlib.components.configuration.ConfigurationItem.item;
import static fr.zcraft.quartzlib.components.configuration.ConfigurationItem.section;

public final class Config extends Configuration
{
    public static final ConfigurationItem<String> TITLE = item("title", "My title");

    public static final DisplayAsSection DISPLAY_AS = section("display-as", DisplayAsSection.class);
    public static class DisplayAsSection  extends ConfigurationSection
    {
        public ConfigurationItem<Boolean> SCOREBOARD = item("scoreboard", true);
        public ConfigurationItem<Boolean> ACTION_BAR = item("action-bar", true);
    }
}
```

The `section` method is used to identify the `DISPLAY_AS` attribute as a section. The string is the section key in the `config.yml` file, and the class is a reference to the sub-class defining the configuration in this section.

The sub-classes have to extend `ConfigurationSection`, and its members are not `static`; excepted that, it's the same as before. You also don't have to mention the parent section in a configuration class (I wrote `item("scoreboard", true);` instead of `item("display-as.scoreboard", true);`).

To access fields defined like this, it's really simple (and readable):

```java
final Boolean displayAsScoreboard = Config.DISPLAY_AS.SCOREBOARD.get();
```

### Advantages of sub-keys references using sub-classes

This kind of class is a bit harder to write, but it comes with some advantages.

1. You can **reuse the configuration sections**.
    See this configuration file (inspired by [BelovedBlocks](https://github.com/zDevelopers/BelovedBlocks/)):

    ```yaml
    oak:
        name: "Oak"
        craftable: true
        itemGlow: true
    birch:
        name: "Oak"
        craftable: true
        itemGlow: true
    # ...
    ```

    You can of course repeat the sections, but you can also write something like this:

    ```java
    public final class Config extends Configuration
    {
        public final BlockSection OAK = section("oak", BlockSection.class);
        public final BlockSection BIRCH = section("birch", BlockSection.class);
        // And 10 more if you want

        static public class BlockSection extends ConfigurationSection
        {
            public final ConfigurationItem<String> NAME = item("name", String.class);
            public final ConfigurationItem<Boolean> CRAFTABLE = item("craftable", true);
            public final ConfigurationItem<Boolean> GLOW = item("itemGlow", true);
        }
    }
    ```

2. You can **extend configuration sections**.
    Now we will use this `config.yml` (inspired by [BelovedBlocks](https://github.com/zDevelopers/BelovedBlocks/) too):

    ```yaml
    oak:
        name: "Oak"
        craftable: true
        itemGlow: true
        amountCrafted: 4
    birch:
        name: "Birch"
        craftable: true
        itemGlow: true
        amountCrafted: 4
    # ...
    stonecutter:
        name: "Stonecutter"
        craftable: true
        itemGlow: true
        usageInLore: true
        percentageBreaking: 0.1
    saw:
        name: "Saw"
        craftable: true
        itemGlow: true
        usageInLore: true
        percentageBreaking: 0.1
    ```

    You can use inheritance mechanisms to write less code and avoid duplicates:

    ```java
    public final class Config extends Configuration
    {
        public final BlockSection OAK = section("oak", BlockSection.class);
        public final BlockSection BIRCH = section("birch", BlockSection.class);
        // And 10 more if you want

        public final ToolSection STONE_CUTTER = section("stonecutter", ToolSection.class);
        public final ToolSection SAW = section("saw", ToolSection.class);
        // Same here

        static public class ItemSection extends ConfigurationSection
        {
            public final ConfigurationItem<String> NAME = item("name", String.class);
            public final ConfigurationItem<Boolean> CRAFTABLE = item("craftable", true);
            public final ConfigurationItem<Boolean> GLOW = item("itemGlow", true);
        }

        static public class BlockSection extends ItemSection
        {
            public final ConfigurationItem<String> AMOUNT_CRAFTED = item("amountCrafted", String.class);
        }

        static public class ToolSection extends ItemSection
        {
            public final ConfigurationItem<Boolean> USAGE_IN_LORE = item("usageInLore", true);
            public final ConfigurationItem<Float> PERCENTAGE_BREAKING = item("percentageBreaking", 0.1f);
        }
    ```

    And from the user point of view, it's still the same.

    ```java
    final float sawPercentage = Config.SAW.PERCENTAGE_BREAKING.get();
    ```

## Lists in the configuration

To retrieve configurations like this:

```yaml
my-list:
  - Item 1
  - Item 2
```

...create an entry like this. Note the `ConfigurationList` instead of `ConfigurationItem`, and the `list` method after.

```java
import static fr.zcraft.quartzlib.components.configuration.ConfigurationItem.list;

public final class Config extends Configuration
{
    public static ConfigurationList<String> MY_LIST = list("my-list", String.class);
}
```

The type parameter of `ConfigurationList<?>` is the data type of the items in the list (here, strings). You also have to give the data type again in the `list` method, due to Java limitations.

The [`ConfigurationList<?>`](https://zdevelopers.github.io/QuartzLib/index.html?fr/zcraft/zlib/components/configuration/ConfigurationList.html) class implements the `List<?>` and `Iterable<?>` interfaces, so you can use them like this:

```java
for (String item : Config.MY_LIST)
    // Do something...
```

## Maps in the configuration

Maps are for YAML structures like this:

```yaml
map:
    key: 1.5
    key2: 45.2
```

where the keys and values can be changed by the user. The keys can be a string or anything that can be extracted from a string. The value can be either a simple value (like the ones of `item`), or a section.

Translation in the code:

```java
public static ConfigurationMap<String, Float> MAP = map("map", String.class, Float.class);
```

Or with a section instead:

```yaml
map:
    key:
        item1: value
        item2: value
    key2:
        item1: value
        item2: value
```

```java
public static ConfigurationMap<String, MapSection> MAP = map("map", String.class, MapSection.class);

public static class MapSection extends ConfigurationSection
{
    public static ConfigurationItem<String> ITEM_1 = item("item1", "value");
    public static ConfigurationItem<String> ITEM_2 = item("item2", "value");
}
```

The [`ConfigurationMap<?, ?>`](https://zdevelopers.github.io/QuartzLib/index.html?fr/zcraft/zlib/components/configuration/ConfigurationMap.html) objects implements the `Map<?,?>` and `Iterable<Map.Entry<?,?>>` interfaces, so you can list the values stored like this:

```java
for (Map.Entry<String, MapSection> entry : Config.MAP.entrySet())
    // Do something...
```

# The great and hidden powers of the Value Handlers

So you have a class to handle your configuration. Nice. You put data types inside. You probably noticed you can use the basic data types just like Bukkit:

```java
public static ConfigurationItem<String> MY_STRING = item("my-string", "default value");
public static ConfigurationItem<Boolean> MY_BOOLEAN = item("my-boolean", true);
public static ConfigurationItem<Double> MY_DOUBLE = item("my-double", 42.69d);
```

But in fact, you can also write this:

```java
public static ConfigurationItem<Material> MY_MATERIAL = item("my-material", Material.NETHER_STAR);
public static ConfigurationItem<Achievement> MY_ACHIEVEMENT = item("my-achievement", Achievement.OPEN_INVENTORY);
```

Or this:

```java
public static ConfigurationItem<ItemStack> MY_ITEM = item("my-item", new ItemStack(Material.STONE, 2));
```

Or even this:

```java
public static ConfigurationItem<MyCustomObject> MY_OBJECT = item("my-object", new MyCustomObject("some default"));
```

This is where things get interesting.

## A Configuration Value Handler?

Behind the scenes, a Configuration Value Handler is what is used to transform a raw value retrieved from the Bukkit configuration accessor to an exploitable, validated value.
It's a simple static method taking a single parameter (retrieved from the config through Bukkit) and returning a parsed and transformed object of the right type (either simple like a `Double` or more complex like an `ItemStack`).

## The built-in handlers

We created a few handlers for usual data types, avoiding you to reimplement them.

<table>
<tr><th><strong>Data type</strong></th><th>Configuration example</th></tr>
<tr>
    <td>
        <strong>All standard data types.</strong>
        <ul>
            <li><code>Boolean</code></li>
            <li><code>Byte</code></li>
            <li><code>Short</code></li>
            <li><code>Integer</code></li>
            <li><code>Long</code></li>
            <li><code>Float</code></li>
            <li><code>Double</code></li>
            <li><code>Char</code></li>
            <li><code>String</code></li>
        </ul>
    </td>
    <td><em>Extracted using the <code>[Type].parse[Type](Object)</code> methods, or <code>toString</code> for strings-related types.</em></td>
</tr>
<tr>
    <td>
        <strong>Any <code>Enum</code>.</strong><br />
        <em>Using <code>Enum.valueOf(String)</code> after dash and spaces replacement by underscores, and upper-casing.</em>
    </td>
    <td><pre lang="yaml">material: DIRT
entity: vex
your-custom-enum: a value</pre></td>
</tr>
<tr>
    <td>
        <strong>A <code>Locale</code>.</strong><br />
        <em>Using <code><a href="http://docs.oracle.com/javase/7/docs/api/index.html?java/util/Locale.Builder.html">Locale.Builder</a>.setLanguageTag(String)</code></em>
    </td>
    <td><pre lang="yaml">lang: fr-FR</pre></td>
</tr>
<tr>
    <td>
        <strong>A Bukkit <code>Vector</code>.</strong><br />
        <em>Use them to retrieve a location.</em>
    </td>
    <td>Either:
<pre lang="yaml">vector: 15,65,-128</pre>
or:
<pre lang="yaml">vector:
    x: 15
    y: 65
    z: -128</pre>
<em>(Fields are optional and default to <code>0</code> in the second format.)</em></td>
</tr>
<tr>
    <td>
        <strong>An <code>Enchantment</code>.</strong><br />
        <em>As they are not technically an <code>Enum</code>.</em>
    </td>
    <td><pre lang="yaml">enchantment: damage_undead</pre></td>
</tr>
<tr>
    <td>
        <strong>An <code>ItemStack</code>.</strong>
    </td>
    <td><pre lang="yaml">item:
    type: cake block  # Only required field
    amount: 12  # Defaults to 0
    data: 1
    title: "The Lie"
    lore:
      - Line 1 of the lore
      - Line 2 of the lore
    glow: true  # Adds a fake enchant to have a glowing item
    hideAttributes: true  # Hides all the item attributes in the tooltip
    enchantments:
        damage_all: 2
        depth_strider: 3
    # Below: only for the Potion type
    effect: fire resistance   # Required
    level: 2   # Defaults to 1
    splash: true   # Defaults to false
    extended: true   # Defaults to false</pre>
<em>(<a href="https://hub.spigotmc.org/javadocs/bukkit/org/bukkit/enchantments/Enchantment.html">Enchantment names</a> — <a href="https://hub.spigotmc.org/javadocs/bukkit/org/bukkit/potion/PotionType.html">Potion effect names</a>)</em></td>
</tr>
<tr>
    <td>
        <strong>A <code>Potion</code>.</strong>
    </td>
    <td><pre lang="yaml">potion:
    effect: fire resistance   # Required
    level: 2   # Defaults to 1
    splash: true   # Defaults to false
    extended: true   # Defaults to false</pre>
<em>(<a href="https://hub.spigotmc.org/javadocs/bukkit/org/bukkit/potion/PotionType.html">Potion effect names</a>)</em></td>
</tr>
<tr>
    <td>
        <strong>A <code>DyeColor</code>.</strong><br />
        <em>It's an enum so you can just use the name, but a support for old numeric codes was added.</em>
    </td>
    <td><pre lang="yaml">dye: yellow
dye: 12</pre>
<em>(<a href="https://hub.spigotmc.org/javadocs/bukkit/org/bukkit/DyeColor.html">Dye names</a>)</em></td>
</tr>
<tr>
    <td>
        <strong>A <code>BannerMeta</code>.</strong>
    </td>
    <td><pre lang="yaml">banner:
    color: black  # A DyeColor, see above
    patterns:
        - {Pattern: hh,Color: 0}
        - {Pattern: cs,Color: 15}
        - {Pattern: hhb,Color: 1}
        - {Pattern: ms,Color: 14}
        - {Pattern: ts,Color: 15}
        - {Pattern: bs,Color: 15}
        - {Pattern: bo,Color: 15}
</pre>
<em>(<a href="http://www.needcoolshoes.com/banner">Banner generator</a>; extract the values from the commands. You can also write the JSON value directly: <code>[{Pattern: hh}, …]</code>.)</em></td>
</tr>
</table>
