# MCI patch

This branch is Spicord 5.8.1 with one change, made by MCIndia Network on 2026-11-21.

- **Base:** upstream tag [`5.8.1`](https://github.com/Spicord/Spicord/tree/5.8.1), commit `e21c6d29431969d828d0145283ce9c435123656b`.
- **Licence:** At this tag Spicord is licensed under the GNU Lesser General Public License v2.1 (see `LICENSE`). This modified version is under the same licence. The change is marked in the source with the date, as LGPL-2.1 section 2(b) requires.

## What changed

In `common/src/main/java/org/spicord/bot/DiscordBot.java`, method `DiscordBot.start()`, the upstream line

```java
//intents.remove(GatewayIntent.GUILD_MEMBERS);
```

is now active (marked `// MCI patch 2026-11-21`). The privileged `GUILD_MEMBERS` (Server Members) gateway intent is left out of the default intent set, the same way upstream already handles `GUILD_PRESENCES`. A loaded addon that lists `GUILD_MEMBERS` in `getRequiredIntents()` still gets it, because `intents.addAll(addon.getRequiredIntents())` runs afterwards. `MESSAGE_CONTENT` and all non-privileged intents are unchanged.

## Why

Our bot only needs the Message Content privileged intent. Unpatched Spicord always requests Server Members too, so Discord's privileged-intent review would have to cover it. With this change the bot can run with Server Members turned off in the Developer Portal.

## Building

```sh
mvn -B clean package    # Maven 3.9.x, run with JDK 21
```

JDK 21 is needed only to run the Velocity API annotation processor. The classes are still compiled for Java 8 (class-file major version 52), the same as upstream. The plugin jar for BungeeCord, Velocity, Bukkit and Sponge is `minecraft/target/Spicord_5.8.1.jar`. It is published as `Spicord-5.8.1-mci1.jar` in release [`5.8.1-mci1`](https://github.com/MCIndiaNetwork/Spicord/releases/tag/5.8.1-mci1). The reported plugin version is still `5.8.1`.
