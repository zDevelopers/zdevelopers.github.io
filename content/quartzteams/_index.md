+++
title = "QuartzTeams"
description = "QuartzTeams — Teams management integrated with QuartzLib"
insert_anchor_links = "right"

[extra]
repo = "https://github.com/zDevelopers/QuartzTeams"
links = [
{ title = "Javadoc", url = "/docs/quartzteams/" }
]
+++

QuartzTeams is a teams management system built on top of QuartzLib, providing:

- team base class with colors, banners, etc.;
- QuartzLib commands ready to be integrated into your plugin;
- GUIs for team selection and management.

There is no tutorial on it right now; please checkout the javadoc in the top left of this page for details.

## Installation


1. Add our Maven repository to your `pom.xml` file.

    ```xml
        <repository>
            <id>zdevelopers-quartzteams</id>
            <url>https://maven.zcraft.fr/QuartzTeams</url>
        </repository>
    ```

2. Add QuartzTeams as a dependency. Be sure to use [the latest version](https://github.com/orgs/zDevelopers/packages?repo_name=QuartzTeams).

    ```xml
        <dependency>
            <groupId>fr.zcraft</groupId>
            <artifactId>quartzteams</artifactId>
            <version>0.0.2</version>
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
                                <include>fr.zcraft:quartzteams</include>
                            </includes>
                        </artifactSet>
                        <relocations>
                            <relocation>
                                <pattern>fr.zcraft.quartzteams</pattern>
                                <shadedPattern>YOUR.OWN.PACKAGE.quartzteams</shadedPattern>
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
