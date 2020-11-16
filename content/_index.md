+++
title = "zDevelopers ⋅ Minecraft plugins and libraries"
+++

We've been making Minecraft plugins for [a ten-years-old old French community](https://zcraft.fr). Over time, we accumulated projets, and tools to create them easily.

Everything we created is published under an open-source licence, usually CeCILL-B (BSD-style in French law), and totally **free to use**.

# Bukkit Plugins

…

# Libraries for developers

To share code between plugins, we created some libraries for Bukkit plugins development. These libraries are open to anyone, and are meant to speed up Minecraft plugins development by implementing common systems or utilities.

## QuartzLib

Formerly zLib, this is our biggest library, by far. Inside, you'll fine composants to handle common Minecraft plugins patterns, like:

- powerful and typed configuration module;
- easier to use commands module;
- simple tools to display rich texts or GUIs;
- natural NBT read and write;
- internationalization;
- scoreboards;
- workers, for any background tasks your plugin may need;
- and lots more.

You'll get each one of these features **without having to ask your users to install a separate lib plugin**, thanks to the magic of Maven shading. Also, only the parts you actually use remain in your plugin JAR at the end, even if QuartzLib as a whole is quite huge.

[**» See all features and how to use it**](/quartzlib)

## zTeams

This way smaller library handles teams in games. We use it for our game plugins, like UHC Reloaded or Balls of Steel. It allows you to:

- create and manage teams and their properties;
- offer pre-built GUIs and commands for players to create (if you allow them to) or join teams.

[**» Check it out**](/zteams)

## Hawk

Hawk is a game report system, typically for competitive Minecraft game but it is very flexible and adapts to a wide range of uses. Integrate it to your game plugin with a few lines of code; then, like an hawk, it'll track everything players do and generate on-demande a beautiful web report. Give  the link to the players so they can see how they performed and share their game with their friends!

You can also add to it special events that may happen during your game.

A [real-life game report](https://hawk.carrade.eu/ZP5Gt2l4) is available, if you want to see what it's like. We provide hosting but you can host a web instance if you prefer to.

[**» Learn how to generate these reports in your games**](/hawk)

## Sentry Bukkit

Sentry-Bukkit is a library to automatically collect crashes and exceptions to a [Sentry](https://sentry.io) project, as well as collecting user feedback from a provided or self-hosted web interface.

_Coming soon._
