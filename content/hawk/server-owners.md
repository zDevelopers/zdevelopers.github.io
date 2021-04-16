+++
title = "Hawk for server owners"
description = "Hawk for server owners — Beautiful reports for your games"

[extra]
menu_title = "For server owners"
+++

Hawk is a game report system, typically for competitive Minecraft game, but it is very flexible and adapts to a wide
range of uses. Run a command on your server; then, like a hawk, it'll track everything players do and generate on-demand
a **beautiful web report**. Give  the link to the players, so they can see how they performed and share their game with
their friends!

As a server owner, the process to create a report is quite simple.

1. At the beginning of the event (or anything), run `/hawk start`. There is a few options available, we'll see that below.
2. Run your event as usual—you can forget Hawk even exist at this point. It records everything in the background.
3. At the end of your event, run `/hawk stop`. After a few seconds, you'll be given a link with the whole game report. Tada! ✨

When Hawk is not recording, it is completely idle, so it won't alter your server performances in any way. And even while
recording, Hawk was optimized to use as less CPU as possible.

_The plugin version has fewer features than the version available to developers. We may make more features available
via commands in the future (e.g. custom events; see [planned features](#planned-features)). Feel free to let us know if
you are interested in such additions._

# Installation

Hawk is a Bukkit plugin. Download it [from SpigotMC](#) or [GitHub](https://github.com/zDevelopers/Hawk/releases/latest)
and install it to your server's `plugins` folder, as usual. Then reboot your server to load the plugin.

There is no configuration file. Everything is done through commands.

# Usage

## Start a report

To start a report, use `/hawk start`. This will start all events recording in the background.

### Giving a name to your report

If you want to name your record, add its name after the command.

```
/hawk start &6&lYour Awesome but Peculiar Event
```

The title can be formatted using
[Minecraft formatting codes](https://minecraft.gamepedia.com/Formatting_codes#Color_codes) with `&` instead of `§`. If
no title is provided, « Minecraft Report » will be used.

### Teams

Hawk support teams. Teams from the server scoreboard will automatically be detected and used in your reports.

### Options

You can add these options after the command to alter Hawk's behaviour.

- `--track-new-players`: by default, only players logged in when you enter the command will be tracked. With this option
  enabled, new players will be tracked as well.
- `--stop-track-on-death`: ask Hawk to stop tracking players when they die. By default, tracking continue but if in your
  game, players have only one life, you don't want to track spectator actions or damages.
- `--stop-track-on-disconnection`: ask Hawk to stop tracking when a player disconnect from the server. By default,
  tracking will continue when they log back in.

As example, to create a report tracking new players but with only one life, called `Time fly`, run:

```
/hawk start --track-new-players Time fly
```

The command will autocomplete these options, so you don't have to memorize them.

While the record is running, you can execute `/hawk info` to get details on the ongoing report.

## Publish the report

To stop the recording and publish the report to the web, run `/haw stop`. After a few seconds, you'll be given a URL
with the report. A file containing all report data will also be saved in the `plugins/Hawk/reports` folder.

If you only want to save the file but don't want the report to be published on the website, run `/hawk stop --no-publish`.

## What if my server crash?

If for some reason you server crash while Hawk is recording, you won't have the option to use `/hawk stop` to publish
the report. But, have no fear! Hawk create regular backups in the `plugins/Hawk/report/backups` folder. These files can
be sent to Hawk to create a report.

As for now, we don't have an easy way to do that, but we plan to add one in the future. In the meantime, if you know how
to do that, you can `POST` the latest backup to `https://hawk.carrade.eu/publish` to have it published.

By default, backups are made every minute, if there were changes since the last backup. Backups are not deleted when you
call `/hawk stop`. Backups are made from another thread, so they will not alter your server's performances.

# Planned features

These are only ideas that would be valuable in the plugin version of Hawk, and which we might implement in the future.
If you need them, [feel free to contribute!](https://github.com/zDevelopers/Hawk)

- Currently, all players from the server are tracked. We plan to add a `--world` option to only track players and
  actions from the world where the command is entered.
- You can only record one report at a time with the Hawk plugin right now. But, the underlying API support multiple
  reports simultaneously. We may add support for that in the plugin in the future.
- We should add a command to publish known backups in case of crash, or any JSON report file.
- We should add a command to create [custom events](/hawk/developers/#adding-events) in reports.
- We should add a commande to [tag players in reports](/hawk/developers/#adding-light-metadata-tags-on-players).
