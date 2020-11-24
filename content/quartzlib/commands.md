+++
title = "Commands | QuartzLib"
description = "Commands"
+++

This component simplifies the management of commands with sub-commands and aliases.

{{ component(doc="https://dev.zcraft.fr/docs/quartzlib/index.html?fr/zcraft/quartzlib/components/commands/package-summary.html", loader="<code>fr.zcraft.quartzlib.components.commands.Commands</code>") }}


# Writing commands

Each sub-command of a command is a class [extending `Command`](http://jenkins.carrade.eu/job/zLib/javadoc/index.html?fr/zcraft/zlib/components/commands/Command.html), with a [`@CommandInfo`](http://jenkins.carrade.eu/job/zLib/javadoc/index.html?fr/zcraft/zlib/components/commands/CommandInfo.html) annotation to describe the command (name, parameters help, aliases). A zero-arguments constructor is required.

In the `Command` sub-class, you have to override the `run()` (and optionally `complete()`) methods, respectively to execute the command and to auto-complete it.

```java
package fr.zcraft.ztoaster.commands;

import fr.zcraft.zlib.components.commands.Command;
import fr.zcraft.zlib.components.commands.CommandException;
import fr.zcraft.zlib.components.commands.CommandInfo;

@CommandInfo(name = "awesomeness")
public class AwesomeCommand extends Command
{
    @Override
    protected void run() throws CommandException
    {
        // Your awesome command here
        sender.sendMessage(ChatColor.GOLD + "Such awesomeness");
        sender.sendMessage(ChatColor.GREEN + "Very grassy");
        sender.sendMessage(ChatColor.GRAY + "Wow.");
    }
}
```

Alongside, a few methods are available to easily retrieve and parse parameters, or to send answers. All the attributes (like `sender` in the example code above) and methods are described in the [documentation of the `Command` class](http://jenkins.carrade.eu/job/zLib/javadoc/index.html?fr/zcraft/zlib/components/commands/Command.html).

Errors are managed with exceptions: if something bad happens, throw a [`CommandException`](http://jenkins.carrade.eu/job/zLib/javadoc/index.html?fr/zcraft/zlib/components/commands/CommandException.html) with the good [reason](http://jenkins.carrade.eu/job/zLib/javadoc/index.html?fr/zcraft/zlib/components/commands/CommandException.Reason.html). Methods are here to do that for you, too.

Method|Description
------|------------
`error(String message)`|To be called if something bad happens.
`throwInvalidArgument(String reason)`|To be called if an argument is invalid (explain why with `reason`).
`playerSender()`|Returns the sender casted to `Player` or throws a `CommandException` with the reason `COMMANDSENDER_EXPECTED_PLAYER`.
`getBooleanParameter(int i)`<br />`getDoubleParameter(int i) `<br />`getFloatParameter(int i)`<br />`getIntegerParameter(int i)`<br />`getLongParameter(int i)`<br />`getEnumParameter(int i, Class<T> type)` |Returns the parameter at the given index casted to the required type, and throws a `CommandException` with the reason `INVALID_PARAMETERS` if it's not possible.
 

# Registering commands

When the subcommands are ready, you'll have to register them like this.

```java
// "toaster" is the name of the root command: /toaster ...
Commands.register("toaster", AwesomeCommand.class, AnotherCommand.class);
```

You can also register shortcut to some subcommands through a simple command; as example, this will execute `/toaster awesomeness` while executing `/awesome` or `/aw`:

```java
Commands.registerShortcut("toaster", AwesomeCommand.class, "awesome", "aw");
```

**Warning!** All the main commands must be registered in the `plugin.yml` file, or an `IllegalStateException` will be thrown.
*Here, `/toaster` and `/awesome` have to be explicitely registered, as `/aw` is registered as an alias.*
 
# Writing documentation

This component auto-generates the user documentation of the commands, through a description when the command is executed without subcommand and an auto-generated `help` subcommand.

## Preview

These preview comes from a real plugin (SpectatorPlus).

* **Command executed without sub-command** (`/command`)
  ![Command executed without sub-command](http://raw.carrade.eu/s/1451482016.png)
* **Sub-commands global help** (`/command help`)
  ![Help sub-command](http://raw.carrade.eu/s/1451482267.png)
* **Detailed help of a sub-command** (`/command help subcommand`)
  ![Help of a command](http://raw.carrade.eu/s/1451485163.png)

## Help files

This said, the documentation does not comes from no-where: the help texts are wrote in a `help` subdirectory of your resources directory (next to the `plugin.yml` file). You can use [`§`-based formatting codes](http://minecraft.gamepedia.com/Formatting_codes) in these files, and new lines are kept.

## `/command`

File location|Content
-------------|-------
—|The message displayed when the command is called without arguments is auto-generated based on:<ul><li>the subcommands (for the « Usage » line in red);</li><li>the `description` field in the `plugin.yml` file (for the other lines).</li></ul>

## `/command help`

File location|Content
-------------|-------
<code>help/<strong>command</strong>.txt</code>|Contains the main help of a command.<br /><ul><li>The first lines contains the introduction, displayed above the commands list.</li><li>The other lines contains the help for each subcommand, following this format.<pre>subcommand:Help of this subcommand</pre>The order of the subcommands is the one in the `Commands.register` method.

## `/command help subcommand`

File location|Content
-------------|-------
<code>help/<strong>command</strong>/<strong>subcommand</strong>.txt</code>|Contains the blue-prefixed help of a subcommand, displayed below the command usage summary.<br />The lines are not automatically truncated to fit well in the chat, so avoid lines above 55-60 characters if you don't want to break the blue line.
