<div align="center">

<img src="assets/icon.png" alt="MasterBridge icon" width="128" height="128">

# MasterBridge

*The complete, modular, ready-to-use Minecraft ↔ Discord bridge*

![Version](https://img.shields.io/badge/version-1.0.0-5865F2)
![Platform](https://img.shields.io/badge/platform-Paper%20%2F%20Spigot%201.20.6%2B-orange)
![Java](https://img.shields.io/badge/java-21-red)

</div>

---

## Table of Contents

1. [Introduction](#1-introduction)
2. [Requirements](#2-requirements)
3. [Installation and first start](#3-installation-and-first-start)
4. [Creating the Discord bot](#4-creating-the-discord-bot)
5. [Discord channels](#5-discord-channels)
6. [Modules (overview)](#6-modules-overview)
7. [In-game commands](#7-in-game-commands)
8. [Discord commands (slash)](#8-discord-commands-slash)
9. [Permissions](#9-permissions)
10. [Account linking](#10-account-linking)
11. [Maintenance](#11-maintenance)
12. [Embed Wizard (/embed)](#12-embed-wizard-embed)
13. [Timed mute](#13-timed-mute)
14. [Command-Spy and Message-Spy](#14-command-spy-and-message-spy)
15. [Security](#15-security)
16. [Placeholders (PlaceholderAPI)](#16-placeholders-placeholderapi)
17. [Message variables](#17-message-variables)
18. [Developer API](#18-developer-api)
19. [Database](#19-database)
20. [FAQ and troubleshooting](#20-faq-and-troubleshooting)

---

## 1. Introduction

**MasterBridge** connects your Minecraft server to Discord with a single plugin: synchronized chat, notifications, slash commands, account linking, schedulable maintenance, mutes, spy tools, Vault economy and much more. Every feature is a **module** you can enable or disable in `config.yml`.

### ✨ Main features

- **Two-way chat sync** with webhook support (player name and avatar on Discord)
- **Notifications** for join/leave, deaths, advancements, PvP kills and staff actions
- **Server online/offline embeds** with optional role ping
- **Live statistics** (players, TPS, RAM, uptime, MOTD) in an automatically updated embed
- **Discord slash commands** to manage the server (kick, ban, whitelist, execute, etc.)
- **Account linking** with LuckPerms → Discord role synchronization
- **Maintenance**, immediate or scheduled, with countdown warnings and history
- **Timed mute**, manageable in game and from Discord
- **Command-Spy / Message-Spy** on Discord and/or in game
- **Alt-account detection** and IP blacklist
- **Embed Wizard**: build and send Discord embeds directly from in game
- **Vault economy** from Discord (`/balance`, `/pay`, `/baltop`)
- **Reaction Actions**: roles or commands triggered by Discord reactions
- **Slow-mode** Discord → Minecraft (also automatic)
- Optional **secondary bot**, scheduled broadcasts, rewards for roles/boosters
- **Public API** and PlaceholderAPI placeholders

---

## 2. Requirements

| Requirement | Details |
|---|---|
| **Server** | Paper (recommended) or Spigot, version 1.20.6 or higher (incl. 1.21.x) |
| **Java** | Java 21 |
| **Discord bot** | A bot application with a token and the privileged intents enabled (see [section 4](#4-creating-the-discord-bot)) |
| **PlaceholderAPI** | Optional: enables the `%masterbridge_*%` placeholders |
| **LuckPerms** | Optional: group → Discord role synchronization |
| **Vault** | Optional: economy commands from Discord |
| **MySQL** | Optional: SQLite is used by default (zero setup). The MySQL driver is not bundled in the jar |

> JDA, HikariCP and SQLite are already bundled (shaded and relocated) in the jar: there is nothing else to install.

---

## 3. Installation and first start

1. Download `MasterBridge-1.0.0.jar` and drop it into the `plugins/` folder.
2. Start the server once: `plugins/MasterBridge/config.yml` will be generated. Then stop the server.
3. Open `config.yml` and fill in at least: `bot.token`, the IDs of the `channels` you need and `modules.discord-commands.guild-id`.
4. Restart the server. The console will show the bot connecting to Discord.
5. Run `/mb test chat` to check that messages reach the channel.

<details>
<summary>Minimal configuration example</summary>

```yaml
bot:
  token: 'YOUR_TOKEN'

channels:
  chat:          '123456789012345678'
  join-leave:    '123456789012345678'

modules:
  discord-commands:
    guild-id: '123456789012345678'
```

</details>

**After every config change** use `/mb reload` (some options, such as the token, require `/mb botrestart`).

---

## 4. Creating the Discord bot

1. Go to the [Discord Developer Portal](https://discord.com/developers/applications) → **New Application**.
2. Open the **Bot** tab → **Reset Token** and copy the token into `bot.token`.
3. Still in **Bot**, enable the **Privileged Gateway Intents**:
   - **Message Content Intent**
   - **Server Members Intent**
4. Open **OAuth2 → URL Generator**: scopes `bot` and `applications.commands`.
5. Recommended permissions: Read/Send Messages, Embed Links, Manage Webhooks, Manage Roles, Add Reactions, Manage Messages.
6. Invite the bot to your server with the generated URL. If you use role sync, the bot's role must sit **above** the roles it has to assign.

> [!WARNING]
> **Never share your bot token.** If you leaked it by mistake, regenerate it immediately from the Developer Portal.

---

## 5. Discord channels

All channels are set in the `channels` section of the config using the channel **ID** (Discord developer mode → right click → Copy ID). You can use the same ID for several channels.

| Key | Purpose |
|---|---|
| `chat` | Minecraft ↔ Discord chat |
| `join-leave` | Player joins and leaves |
| `death` | Deaths |
| `staff-log` | Staff log (ban, kick, mute) |
| `console` | Console relay **(admins only!)** |
| `stats` | Live statistics embed |
| `pvp` | PvP kill feed |
| `rewards` | Rewards |
| `advancement` | Advancements earned |
| `link` | Account link notifications |
| `server-status` | Server online/offline embeds |
| `maintenance` | Maintenance notifications |
| `command-log` | Executed commands log |
| `mutes` | Mute/unmute notifications |
| `alt-alerts` | Alt-account alerts |
| `command-spy` | Command-spy output (if output is `DISCORD`/`BOTH`) |
| `message-spy` | Message-spy output (if output is `DISCORD`/`BOTH`) |

---

## 6. Modules (overview)

Each module lives under `modules.<name>` and has an `enabled` switch.

| Module | Default | Description |
|---|:---:|---|
| `chat-sync` | ✅ | Syncs the chat. Direction `BOTH` / `MC_TO_DISCORD` / `DISCORD_TO_MC`, webhooks, custom formats, ignores bots |
| `join-leave` | ✅ | Join/leave/first-join messages, embed or text, with player counter |
| `server-status` | ✅ | Embeds on server start/stop, with thumbnail and role ping |
| `death` | ✅ | Death messages |
| `advancement` | ✅ | Advancement notifications |
| `staff-log` | ✅ | Ban, kick and mute log with custom formats |
| `discord-commands` | ✅ | Discord slash commands, each one toggleable and restrictable to one or more roles |
| `live-stats` | ✅ | Embed refreshed every N seconds (TPS, RAM, uptime, MOTD) |
| `account-link` | ✅ | Code-based account linking, automatic role, `require-link` option |
| `pvp-feed` | ✅ | Kill feed with weapon and distance |
| `anti-spam` | ✅ | Cooldown and message limit, separate for Discord and Minecraft |
| `word-filter` | ✅ | Shared word filter, `BLOCK` or `REPLACE` action |
| `maintenance` | ✅ | Maintenance mode (see [section 11](#11-maintenance)) |
| `embed-sender` | ✅ | `/embed` wizard to send embeds from in game |
| `avatar-sync` | ✅ | Skin avatar in embeds (`CRAFATAR`, `MC_HEADS`, `MINOTAR`) |
| `role-sync` | ❌ | LuckPerms groups → Discord roles |
| `rewards` | ❌ | Commands/rewards based on Discord role or booster status |
| `console-relay` | ❌ | Server console in a Discord channel, with allowed roles |
| `scheduled-broadcasts` | ❌ | Periodic messages on Discord and (optionally) in game |
| `vault-economy` | ❌ | `/balance`, `/pay`, `/baltop` from Discord |
| `command-log` | ❌ | Command log, `STAFF` or `ALL` mode, with an ignore list |
| `mute-system` | ❌ | Timed mute with countdown warnings |
| `alt-detection` | ❌ | Alt-account detection and IP blacklist |
| `reaction-actions` | ❌ | Discord reactions → role or command |
| `slow-mode` | ❌ | Discord → Minecraft slow-mode, manual or automatic |
| `secondary-bot` | ❌ | Optional second Discord bot |
| `command-spy` | ❌ | Spies on player commands |
| `message-spy` | ❌ | Spies on private messages |

### Other config sections

| Section | Content |
|---|---|
| `bot` | Token, activity type (`PLAYING`, `WATCHING`, `LISTENING`, `COMPETING`, `STREAMING`), text (supports `{online}` and `{max}`), status and update interval |
| `database` | `SQLITE` or `MYSQL` type and related settings |
| `messages` | Prefix and system messages (link, permissions, reload, etc.) |
| `formatting` | Color stripping towards Discord, markdown → MC colors conversion, maximum lengths, embed footer |
| `debug` / `check-updates` | Debug logging and update checking |

<details>
<summary>Example: chat sync with webhooks</summary>

```yaml
modules:
  chat-sync:
    enabled: true
    direction: BOTH
    format-mc-to-discord: '**{player}**: {message}'
    format-discord-to-mc: '&7[&5Discord&7] &f{tag}&7: &f{message}'
    use-webhooks: true
    webhook-url: ''        # empty = created automatically
    mention-roles: false
    mention-everyone: false
    ignore-bots: true
```

</details>

> [!NOTE]
> The default texts in the shipped `config.yml` are in Italian. You can translate every message, embed and format to any language you like.

---

## 7. In-game commands

### Main command: `/masterbridge`

Aliases: `/mb`, `/bridge` — Permission: `masterbridge.admin`

| Command | Description |
|---|---|
| `/mb` | Shows help |
| `/mb reload` | Reloads the configuration |
| `/mb status` | Bridge and bot status |
| `/mb info` | Plugin information |
| `/mb modules` | Lists the modules and their state |
| `/mb debug` | Toggles debug mode |
| `/mb botstart` | Starts the Discord bot |
| `/mb botstop` | Stops the bot (needs confirmation) |
| `/mb botrestart` (or `restartbot`) | Restarts the bot (needs confirmation) |
| `/mb confirm` / `/mb cancel` | Confirms or cancels the pending action. Confirmation expires after 15 seconds |
| `/mb test <chat\|join\|leave\|death\|pvp\|staff>` | Sends a test notification to the matching channel |
| `/mb slowmode <on\|off\|status>` | Manages Discord → Minecraft slow-mode |
| `/mb secondbot <start\|stop\|status>` | Manages the secondary bot |
| `/mb commandspy <on\|off\|status>` | Manages command-spy |
| `/mb msgspy <on\|off\|status>` | Manages message-spy |

### Player commands

| Command | Permission | Description |
|---|---|---|
| `/discord [link\|unlink\|info]` | — | Link/unlink your account and show the bridge status |
| `/discordlink` (alias `/mlink`) | `masterbridge.link` | Generates the code to link your Discord account |

### Staff commands

| Command | Permission | Description |
|---|---|---|
| `/checklink <player>` | `masterbridge.staff` | Checks whether a player is linked to Discord |
| `/maintenance` (aliases `/maint`, `/mt`) | `masterbridge.maintenance` | Maintenance management ([section 11](#11-maintenance)) |
| `/embed` | `masterbridge.embed` | Starts the embed wizard ([section 12](#12-embed-wizard-embed)) |
| `/mute <player> [duration] [reason]` | `masterbridge.mute` | Mutes a player |
| `/unmute <player>` | `masterbridge.mute` | Removes a mute |
| `/muteinfo <player>` | `masterbridge.mute` | Shows mute information |

---

## 8. Discord commands (slash)

Enable them in `modules.discord-commands` (the `guild-id` is required). For commands with `required-role` you can set a single role or a list with `required-roles` (having any one is enough). If no role is set, the command is available to everyone.

| Command | Options | Description |
|---|---|---|
| `/online` | — | Online players |
| `/tps` | — | Server TPS and statistics |
| `/say` | `message` | Sends a message to Minecraft chat |
| `/kick` | `player`, `reason` (opt.) | Kicks a player |
| `/ban` | `player`, `reason` (opt.) | Bans a player |
| `/unban` | `player` (name or UUID) | Removes a ban |
| `/whitelist` | `action` (add/remove), `player` | Manages the whitelist |
| `/execute` | `command` | Runs a console command ⚠️ **admins only!** |
| `/link` | `code` | Links your Minecraft account with the code received in game |
| `/unlink` | — | Unlinks your account |
| `/whois` | `player` (opt., MC name or Discord ID) | Info about a linked player |
| `/checklink` | `player` | Checks whether a player is linked (staff) |
| `/mute` | `player`, `duration` (opt.), `reason` (opt.) | Mutes a player (requires `mute-system`) |
| `/unmute` | `player` | Unmutes a player |
| `/commandspy` | `state` (on/off/status) | Manages command-spy |
| `/msgspy` | `state` (on/off/status) | Manages message-spy |
| `/balance` | — | Your balance (requires `vault-economy`) |
| `/pay` | `user`, `amount` | Sends money to a linked player |
| `/baltop` | — | Top balances leaderboard (configurable limit) |

> `/link`, `/unlink`, `/whois` and `/checklink` are registered when the `account-link` module is enabled; `/balance`, `/pay` and `/baltop` when `vault-economy` is enabled; `/mute` and `/unmute` when `mute-system` has `discord-commands: true`; `/commandspy` and `/msgspy` when the respective modules have `discord-toggle-command: true`.

---

## 9. Permissions

| Permission | Default | Description |
|---|:---:|---|
| `masterbridge.admin` | OP | Full admin access. Includes staff, maintenance, mute and spy |
| `masterbridge.staff` | OP | Staff notifications (link, actions from Discord) and `/checklink` |
| `masterbridge.maintenance` | OP | Manage maintenance mode |
| `masterbridge.maintenance.bypass` | OP | Join the server during maintenance |
| `masterbridge.mute` | OP | Use `/mute`, `/unmute`, `/muteinfo` |
| `masterbridge.bypass.mute` | OP | Cannot be muted / can chat while muted |
| `masterbridge.spy` | OP | Receives command-spy and message-spy notifications in game |
| `masterbridge.link` | Everyone | Link your own Discord account |
| `masterbridge.embed` | OP | Use the `/embed` wizard |
| `masterbridge.bypass.filter` | OP | Bypasses the word filter |
| `masterbridge.bypass.antispam` | OP | Bypasses anti-spam |

---

## 10. Account linking

1. The player runs `/mlink` (or `/discord link`) in game and receives a code.
2. On Discord they run `/link <code>` before the code expires.
3. The account is linked: "linked" role assigned, role sync (if enabled), staff notified.

| Option (`modules.account-link`) | Description |
|---|---|
| `code-length` | Code length (default `6`) |
| `code-expire-minutes` | Code expiry in minutes (default `10`) |
| `auto-role-sync` | Syncs roles after linking |
| `require-link` | If `true`, unlinked players receive a reminder (`require-link-message`) shortly after joining. They are **not** blocked |
| `notify-staff-on-link` | Notifies staff when an account gets linked |
| `staff-notify-permission` | Permission to receive the notification (default `masterbridge.staff`) |

### Role synchronization (LuckPerms → Discord)

Enable `modules.role-sync` and map LuckPerms groups to Discord roles:

```yaml
role-sync:
  enabled: true
  guild-id: '123456789012345678'
  mappings:
    - minecraft-group: vip
      discord-role: '111111111111111111'
    - minecraft-group: admin
      discord-role: '222222222222222222'
  linked-role: '333333333333333333'   # given to every linked account
```

---

## 11. Maintenance

Command: `/maintenance` (aliases `/maint`, `/mt`) — permission `masterbridge.maintenance`

| Command | Description |
|---|---|
| `/maintenance on [duration] [reason]` | Enables it right away. The duration (e.g. `30m`) is an optional estimate |
| `/maintenance off` | Ends maintenance |
| `/maintenance schedule <delay> [duration] [reason]` | Schedules activation, with countdown warnings |
| `/maintenance cancel` | Cancels the scheduled maintenance |
| `/maintenance status` | Current status |
| `/maintenance history [n]` | History of the latest maintenances |

**Duration format:** combinations of `h` (hours), `m` (minutes), `s` (seconds). Examples: `90s`, `30m`, `1h`, `2h30m`.

**Countdown warnings** (in game and on Discord) for scheduled maintenance: 1 hour, 30, 15, 5 and 1 minute before, whenever the delay is long enough.

### Behavior on activation (`kick-on-enable`)

| Value | Effect |
|---|---|
| `DISABLED` | Players are left alone (you can use `commands-on-enable`) |
| `KICK` (default) | Players are kicked with `kick-message`, including those trying to join |
| `TRANSFER` | Players are transferred to another BungeeCord server (`transfer-server`). Requires BungeeCord |

Other useful options: `commands-on-enable` / `commands-on-disable` (run by the console, with `{executor}`), `discord-auto-reply` (the bot answers in chat during maintenance), customizable embeds (title, description, color, thumbnail, image) and `ping-role`. Players with `masterbridge.maintenance.bypass` can still join.

---

## 12. Embed Wizard (`/embed`)

Permission: `masterbridge.embed`. The wizard guides you step by step, in chat, through building an embed to send to Discord. Type `skip` to skip optional steps (you can also click the buttons in chat) and `cancel` (or `annulla`) to abort.

1. **Channel**: chosen among those defined in `embed-sender.channels`
2. **Title**
3. **Description**
4. **Color** (skip = default color)
5. **Footer** (skip = no footer)
6. **Thumbnail** (skip)
7. **Image** (skip)
8. **Extra fields**: name, value and inline flag, repeatable
9. **Preview**: confirm sending or cancel

```yaml
embed-sender:
  enabled: true
  channels:
    announcements: '123456789012345678'
    updates:       '123456789012345678'
    staff:         '123456789012345678'
```

The key (e.g. `announcements`) is the name shown in the wizard.

---

## 13. Timed mute

Enable `modules.mute-system`. Commands: `/mute`, `/unmute`, `/muteinfo` (and the slash equivalents on Discord).

**Durations:** `s` seconds, `m` minutes, `h` hours, `g` (or `d`) days, `perm` = permanent.
Examples: `/mute Steve 30m Spam`, `/mute Steve 1d Insults`, `/mute Steve perm`.
Without a duration, `default-duration-seconds` (`600`) is used.

| Option | Description |
|---|---|
| `default-duration-seconds` | Default duration |
| `check-interval-seconds` | How often expirations and countdowns are checked |
| `countdown-warnings-seconds` | Warns the player when N seconds are left (e.g. `[60, 10]`) |
| `log-to-discord` | Posts mute/unmute to the `mutes` channel |
| `discord-commands` | Enables `/mute` and `/unmute` from Discord |
| `mute-message`, `unmute-message`, `countdown-message`, `chat-blocked-message` | Customizable messages. Variables: `{staff}`, `{duration}`, `{reason}`, `{seconds}`, `{remaining}` |

Mutes are stored in the database and survive restarts. Players with `masterbridge.bypass.mute` cannot be muted.

---

## 14. Command-Spy and Message-Spy

Two similar modules to monitor player commands and private messages in real time.

| Aspect | Command-Spy | Message-Spy |
|---|---|---|
| **What it shows** | All executed commands | Private messages between players |
| **Activation** | `default-state`, `/mb commandspy`, `/commandspy` from Discord | `default-state`, `/mb msgspy`, `/msgspy` from Discord |
| **Output** | `DISCORD`, `INGAME` or `BOTH` | `DISCORD`, `INGAME` or `BOTH` |
| **Command list** | `ignored-commands`: never spied on (login, register, etc.) | `watched-commands`: commands treated as private messages (msg, tell, w, r, ...) |
| **In-game permission** | `masterbridge.spy` | `masterbridge.spy` |

> [!IMPORTANT]
> Adapt `watched-commands` to your server's messaging plugin (EssentialsX, etc.) and keep every password-related command in `ignored-commands`. Inform your players if you log commands and messages, in line with the privacy rules that apply to you.

---

## 15. Security

### Anti-spam

| Option | Default | Description |
|---|:---:|---|
| `discord-cooldown-ms` | `2000` | Minimum pause between two Discord messages |
| `discord-max-messages` / `discord-time-window-s` | `5` / `10` | Max messages within the time window (Discord) |
| `mc-cooldown-ms` | `1500` | Minimum pause between two in-game messages |
| `mc-max-messages` / `mc-time-window-s` | `4` / `8` | Max messages within the time window (Minecraft) |

### Word filter

`action: BLOCK` blocks the message (on Discord the bot adds a 🚫 reaction), `REPLACE` replaces letters with `replace-char`. With `shared: true` the filter applies to both Discord and Minecraft.

### Alt-account detection and IP blacklist

With `alt-detection` the plugin records player IPs and, when a new IP is shared with another account, sends an alert to `alt-alerts` and (optionally) to staff in game. With `mask-ip-in-alert` the IP is masked (e.g. `192.168.*.*`). The `blacklist` denies entry to the listed IPs and shows the `kick-message`.

---

## 16. Placeholders (PlaceholderAPI)

With PlaceholderAPI installed the plugin registers the `masterbridge` expansion.

| Placeholder | Result |
|---|---|
| `%masterbridge_discord_online%` | Online members on the Discord server |
| `%masterbridge_discord_members%` | Total members of the Discord server |
| `%masterbridge_bot_status%` | Online / Offline |
| `%masterbridge_linked%` | `true` / `false`: the player has linked Discord |
| `%masterbridge_discord_tag%` | Linked Discord tag, or `-` |
| `%masterbridge_muted%` | `true` / `false`: the player is muted |
| `%masterbridge_mute_remaining%` | Remaining mute time, `permanente` (permanent) or `-` |
| `%masterbridge_balance%` | Player's Vault balance (`0` if Vault is not active) |

---

## 17. Message variables

| Context | Variables |
|---|---|
| Bot activity | `{online}`, `{max}` |
| Chat sync | `{player}`, `{message}`, `{tag}` |
| Join / Leave | `{player}` |
| Death | `{death_message}` |
| Advancement | `{player}`, `{advancement}` |
| PvP feed | `{killer}`, `{victim}`, `{weapon}`, `{distance}` |
| Staff log | `{staff}`, `{player}`, `{reason}` |
| Command-log / Command-spy | `{player}`, `{command}` |
| Message-spy | `{sender}`, `{target}`, `{message}` |
| Account link | `{code}`, `{minutes}`, `{tag}` |
| Mute | `{staff}`, `{duration}`, `{reason}`, `{seconds}`, `{remaining}` |
| Maintenance | `{reason}`, `{executor}` |
| Rewards and reaction actions | `{player}` |

**Colors:** in messages sent to Minecraft use `&` codes (e.g. `&a`, `&l`); messages sent to Discord use markdown (`**bold**`, `` `code` ``).

---

## 18. Developer API

Other plugins can use MasterBridge through the `MasterBridgeAPI` interface (package `it.mastercraft.masterbridge.api`).

```java
RegisteredServiceProvider<MasterBridgeAPI> rsp =
        Bukkit.getServicesManager().getRegistration(MasterBridgeAPI.class);
if (rsp != null) {
    MasterBridgeAPI api = rsp.getProvider();
    api.sendMessage("chat", "Hello from another plugin!");
}
```

Alternatively: `MasterBridge.getInstance().getAPI()`. Remember to add `MasterBridge` to your plugin's `depend` or `softdepend`.

| Method | Description |
|---|---|
| `boolean isDiscordConnected()` | `true` if the main bot is connected |
| `void sendMessage(String channelKey, String message)` | Sends a message to a configured channel (key from `channels.*`) |
| `void sendEmbed(String channelKey, String title, String description, String hexColor)` | Sends a simple embed (color e.g. `"00FF00"`) |
| `boolean isLinked(UUID uuid)` | Does the player have a linked Discord account? |
| `String getDiscordTag(UUID uuid)` | Discord tag, or `null` |
| `String getDiscordId(UUID uuid)` | Discord ID, or `null` |
| `boolean isMuted(UUID uuid)` | Is the player muted? |
| `long getMuteRemainingSeconds(UUID uuid)` | Seconds left (`-1` permanent, `0` not muted) |
| `void mutePlayer(UUID, String name, String reason, String mutedBy, long seconds)` | Mutes a player (`seconds <= 0` = permanent) |
| `boolean unmutePlayer(UUID, String name, String unmutedBy)` | Unmutes; `true` if the player was muted |
| `boolean isEconomyAvailable()` | Is Vault economy available? |
| `double getBalance(UUID uuid)` | Vault balance (`0` if unavailable) |
| `boolean sendMessageViaSecondaryBot(String channelId, String message)` | Sends through the secondary bot (raw channel ID) |

---

## 19. Database

| Type | Notes |
|---|---|
| **`SQLITE`** (default) | Zero setup: creates `masterbridge.db` in the plugin folder |
| **`MYSQL`** | Set `host`, `port`, `database`, `username`, `password` and `pool-size` under `database.mysql` |

The plugin stores linked accounts, pending codes, reward logs, maintenance history, mutes and IP history.

---

## 20. FAQ and troubleshooting

<details>
<summary><b>The bot does not connect</b></summary>

Check that the token is correct and has no spaces, that the **Message Content** and **Server Members** intents are enabled in the Developer Portal, and that the server has Internet access. Set `debug: true` and read the console.

</details>

<details>
<summary><b>Slash commands do not show up</b></summary>

Make sure `modules.discord-commands.guild-id` is the correct ID, that the bot was invited with the `applications.commands` scope and that the module is enabled. After changing it, use `/mb botrestart`.

</details>

<details>
<summary><b>Messages don't reach Discord</b></summary>

Check the ID in `channels.chat`, the bot's permissions on the channel and that `chat-sync.direction` is `BOTH` or `MC_TO_DISCORD`. Try `/mb test chat`.

</details>

<details>
<summary><b>Discord messages don't reach the game</b></summary>

Make sure the **Message Content Intent** is enabled, the direction is `BOTH` or `DISCORD_TO_MC` and that `ignore-bots` isn't discarding messages you expect.

</details>

<details>
<summary><b>Role sync doesn't assign roles</b></summary>

The bot's role must be above the roles to assign, the bot needs the **Manage Roles** permission and the player must have linked their account. LuckPerms and `role-sync.enabled: true` are also required.

</details>

<details>
<summary><b>Can I translate the messages?</b></summary>

Yes. All texts (messages, embeds, formats) are in `config.yml` and you can write them in any language you prefer.

</details>

<details>
<summary><b>How do I report a bug or request a feature?</b></summary>

Open an [issue](../../issues) on this repository, including your server version, plugin version and any console error (use a code block or a paste service).

</details>

---

<div align="center">

*Thank you for choosing MasterBridge! ❤️*
If you enjoy the plugin, please leave a ⭐ on the repository.

</div>
