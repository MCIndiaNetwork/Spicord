# MCI patch

This branch is Spicord 5.7.2 with one change, made by MCIndia Network on 2026-11-21.

- **Base:** upstream tag [`5.7.2`](https://github.com/Spicord/Spicord/tree/5.7.2), commit `f9f0bf96d09cc688997b3a3fce049bc95e712af7`.
- **Licence:** Spicord 5.7.2 is licensed under the GNU Affero General Public License v3.0 or later (see `LICENSE` and the file headers). This modified version is under the same licence. Upstream moved to LGPL-2.1 in Feb 2026, but that applies only to later releases, not to this base.

## What changed

In `common/src/main/java/org/spicord/bot/DiscordBot.java`, method `DiscordBot.start()`, the upstream line

```java
//intents.remove(GatewayIntent.GUILD_MEMBERS);
```

is now active (marked `// MCI patch 2026-11-21`). The privileged `GUILD_MEMBERS` (Server Members) gateway intent is left out of the default intent set, the same way upstream already handles `GUILD_PRESENCES`. A loaded addon that lists `GUILD_MEMBERS` in `getRequiredIntents()` still gets it, because `intents.addAll(addon.getRequiredIntents())` runs afterwards. `MESSAGE_CONTENT` and all non-privileged intents are unchanged.

## Why

Our bot only needs the Message Content privileged intent. Unpatched Spicord always requests Server Members too, so Discord's privileged-intent review would have to cover it. With this change the bot can run with Server Members turned off in the Developer Portal.

## Binary

Release [`5.7.2-mci1`](https://github.com/MCIndiaNetwork/Spicord/releases/tag/5.7.2-mci1) has `Spicord-5.7.2-mci1.jar`. It is the upstream 5.7.2 jar with only the six `org/spicord/bot/DiscordBot*.class` files replaced by ones compiled from this source (javac 17, `--release 8`, Lombok 1.18.36). Every other entry is byte-identical, and the reported plugin version is still `5.7.2`.
