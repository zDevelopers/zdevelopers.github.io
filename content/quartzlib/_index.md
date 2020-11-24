+++
title = "QuartzLib"
description = "QuartzLib"
insert_anchor_links = "right"

[extra]
repo = "https://github.com/zDevelopers/QuartzLib"
links = [
    { title = "Javadoc", url = "/docs/quartzlib" },
    { title = "Spigot", url = "https://www.spigotmc.org/resources/quartzlib.32310/" }
]
+++

QuartzLib (formerly zLib) is a helper library for Bukkit plugins development, providing useful tools for small and big tasks, to fill the gaps of the Bukkit API.

# Installation

You'll need approximately two minutes to integrate the QuartzLib inside your plugin, if you are using Maven already.

Currently, QuartzLib requires **Java 8** or later, **Bukkit 1.15** or later. This document explains how to install QuartzLib if you are using **Maven**, but it works too with other system, as long as a shading-like feature is present.
 

## Integration of the library files inside your plugin

1. Add our Maven repository to your `pom.xml` file.

    ```xml
        <repository>
            <id>zdevelopers-quartzlib</id>
            <url>https://maven.zcraft.fr/QuartzLib</url>
        </repository>
    ```

2. Add QuartzLib as a dependency.

    ```xml
        <dependency>
            <groupId>fr.zcraft</groupId>
            <artifactId>quartzlib</artifactId>
            <version>0.0.1-SNAPSHOT</version>
        </dependency>
    ```

3. Add the shading plugin to the build. Replace **`YOUR.OWN.PACKAGE`** with your own package.

   ```xml
        <build>
            ...
            <plugins>
                ...
                <plugin>
                    <groupId>org.apache.maven.plugins</groupId>
                    <artifactId>maven-shade-plugin</artifactId>
                    <version>2.4</version>
                    <configuration>
                        <artifactSet>
                            <includes>
                                <include>fr.zcraft:quartzlib</include>
                            </includes>
                        </artifactSet>
                        <relocations>
                            <relocation>
                                <pattern>fr.zcraft.quartzlib</pattern>
                                <shadedPattern>YOUR.OWN.PACKAGE</shadedPattern>
                            </relocation>
                        </relocations>
                    </configuration>
                    <executions>
                        <execution>
                            <phase>package</phase>
                            <goals>
                                <goal>shade</goal>
                            </goals>
                        </execution>
                    </executions>
                </plugin>
                ...
            </plugins>
            ...
        </build>
   ```

4. Build your project as usual, as example with the following command from your working directory, or using an integrated tool from your IDE.

   ```bash
   mvn clean install
   ```


## Library & components auto-loading

Currently, there is only one simple way to integrate this library. The QuartzLib provides [his own version](https://zdevelopers.github.io/QuartzLib/?fr/zcraft/quartzlib/core/QuartzPlugin.html) of the `JavaPlugin` main class, to semi-automatically load and unload the QuartzLib components and core.

1. In your main class, replace `extends JavaPlugin` with `extends QuartzPlugin` (in `fr.zcraft.quartzlib.core.QuartzPlugin`).

    ```java
    import fr.zcraft.quartzlib.core.QuartzPlugin;

    public class Toaster extends QuartzPlugin
    {
        // ...
    }
    ```

2. Use inside `onEnable()` the [`loadComponents`](https://zdevelopers.github.io/QuartzLib/fr/zcraft/quartzlib/core/QuartzPlugin.html#loadComponents-java.lang.Class...-) method to load the components you will use.

    ```java
    @Override
    public void onEnable()
    {
        loadComponents(Gui.class, Commands.class, SidebarScoreboard.class);
    }
    ```

The `loadComponents` method is the standard way to load the classes (components) used in your plugin. It's a generic tool to load QuartzLib components (GUIs, sidebars...), but also your owns.
In QuartzLib components wiki pages, the component class to load through this method will always be given.


#### A note about overridden methods

The QuartzLib overrides the `onLoad` method.
If your plugin uses it you **have to** call `super.onLoad()` like this.

```java
@Override
public void onLoad()
{
    super.onLoad();

    // Your code, placed after.
}
```
