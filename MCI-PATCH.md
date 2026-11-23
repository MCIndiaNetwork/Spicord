# MCI patch

This branch is Spicord 5.8.1 with two one-line changes by MCIndia Network, made on 2026-11-21 and 2026-11-23. With both changes, Spicord requests **no privileged gateway intents by default**.

- **Base:** upstream tag [`5.8.1`](https://github.com/Spicord/Spicord/tree/5.8.1), commit `e21c6d29431969d828d0145283ce9c435123656b`.
- **Licence:** At this tag Spicord is licensed under the GNU Lesser General Public License v2.1 (see `LICENSE`). This modified version is under the same licence. Each change is marked in the source with its date, as LGPL-2.1 section 2(b) requires.

## What changed

In `common/src/main/java/org/spicord/bot/DiscordBot.java`, method `DiscordBot.start()`, two upstream lines that were commented out are now active:

```java
intents.remove(GatewayIntent.GUILD_MEMBERS);   // MCI patch 2026-11-21 (release 5.8.1-mci1)
intents.remove(GatewayIntent.MESSAGE_CONTENT); // MCI patch 2026-11-23 (release 5.8.1-mci2)
```

Upstream already removes `GUILD_PRESENCES`, so with these two lines all three privileged intents (Presence, Server Members, Message Content) are left out of the default intent set. A loaded addon that needs one lists it in `getRequiredIntents()` (or `requiredIntents` for a JavaScript addon), and it is added back, because `intents.addAll(addon.getRequiredIntents())` runs afterwards. The non-privileged intents are unchanged.

## Why

Our bot uses only slash commands, so it needs no privileged intents. Discord's privileged-intent review then doesn't need to cover any of them, and Server Members and Message Content can both be turned off in the Developer Portal.

## What needs Message Content

Slash commands and autocomplete do not need it. The built-in addons `spicord::info`, `spicord::players` and `spicord::plugins` use only slash commands.

Without Message Content, Discord sends empty text for guild messages from other users that don't mention the bot. As a result:

- **Legacy prefix commands do not work in guild channels.** These are commands like `!cmd`, registered with the deprecated `DiscordBot.onCommand(...)` or an addon's `commands` array. Because the text is empty, the prefix never matches. They still work in DMs and in messages that mention the bot.
- **Addons that read message text in `onMessageReceived` get empty content** for those messages.
- **JDA logs one warning** the first time Spicord's listener reads such a message: `Attempting to access message content without GatewayIntent.MESSAGE_CONTENT...`. It appears once per JVM and is harmless.

An addon that really needs message text should return `MESSAGE_CONTENT` from `getRequiredIntents()`. The intent must then also be enabled in the Developer Portal.

## Keep `command_support = true`

Do not set `command_support = false` in `config.toml`. Slash commands and autocomplete are dispatched by the same listener that handles prefix commands, `BotCommandListener` (`DiscordBot.java`, around lines 710-775). `DiscordBot` only registers that listener when command support is on (around line 180).

If command support is off, slash commands are still registered with Discord but never answered, and no addon's `onMessageReceived` is called. `command_prefix` must also be non-empty, or Spicord turns command support off by itself (around line 123). This patch does not change that logic.

## Building

```sh
mvn -B clean package    # Maven 3.9.x, run with JDK 21
```

JDK 21 is needed only to run the Velocity API annotation processor. The classes are still compiled for Java 8 (class-file major version 52), the same as upstream. The plugin jar for BungeeCord, Velocity, Bukkit and Sponge is `minecraft/target/Spicord_5.8.1.jar`. It is published as `Spicord-5.8.1-mci2.jar` in release [`5.8.1-mci2`](https://github.com/MCIndiaNetwork/Spicord/releases/tag/5.8.1-mci2). The reported plugin version is still `5.8.1`. Release [`5.8.1-mci1`](https://github.com/MCIndiaNetwork/Spicord/releases/tag/5.8.1-mci1) has only the `GUILD_MEMBERS` change.
