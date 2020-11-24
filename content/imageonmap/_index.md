+++
title = "ImageOnMap"
description = "ImageOnMap — Display any image in Minecraft"
insert_anchor_links = "right"

[extra]
repo = "https://github.com/zDevelopers/ImageOnMap"
links = [
    { title = "Spigot", url = "https://www.spigotmc.org/resources/imageonmap.26585/" },
    { title = "BukkitDev", url = "https://dev.bukkit.org/projects/imageonmap" }
]
+++

ImageOnMap is a Bukkit plugin made to allow players to display any image (in PNG, JPEG or GIF format) directly in game, allowing to enhance their builds and add unique assets to them!

ImageOnMap use maps (hence the name), but instead of the terrain, you see your image. Put the map into an item frame and everyone can enjoy it too!

Sometimes, images are too big to fit in a single Minecraft map. In this case, ImageOnMap automatically cut the image in as many parts as needed, so your image can be displayed into its full glory. And have no fear of the headache of placing all individual maps into a wall of item frames! ImageOnMap automatically deploy your poster as long as the frame are placed on the wall or the ceiling.

_**Note** — We cannot display any colour on a Minecraft map; only a limited subset of colours are usable. As a consequence, images may not look exactly the same in Minecraft._

# Downloads and installation

To get started, [download the plugin JAR for your Minecraft version](#) and put it into your `plugins` directory. Then reboot your server.

# How to use ImageOnMap

To render an image, use the `/tomap` command, followed by the direct link (URL) to your image. As example, if you want to render in game [this beautiful Klein bottle](https://upload.wikimedia.org/wikipedia/commons/2/21/Acme_klein_bottle.jpg), you'll type:

```
/tomap https://upload.wikimedia.org/wikipedia/commons/2/21/Acme_klein_bottle.jpg
```

As a result, ImageOnMap will offer a special shiny map with all your image in it.

## Resizing images

Simple but later.
