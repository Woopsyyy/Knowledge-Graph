# Himawari Bot — Architecture

Part of [[Himawari-System]]. The Discord side: account linking + member renaming, a ticket system,
an embed builder, and greet/leave/boost messages.

- **Stack**: Node.js, `discord.js` 14 (`@discordjs/builders`, `@discordjs/rest`,
  `@supabase/supabase-js`). Intents: `Guilds`, `GuildMembers`.
- **Config**: `Himawari Bot/config.json` (`TOKEN`, `SUPABASE_URL`, `SUPABASE_SERVICE_KEY`,
  `GUILD_ID`, linked-role/message settings). Deploys to **Discloud** (`discloud.config`).

## Runtime wiring (`index.js`)
1. Create `Client`; attach `client.config`, `client.cooldowns`, `client.cache`.
2. `utils/Supabase.js(client)` — attaches the lazy Supabase data layer as `client.db`.
3. `utils/ComponentLoader.js` — folder-walks `commands/`, `buttons/`, `dropdowns/`, `modals/` into
   `client[type]` maps (via `ReadFolder.js`).
4. `utils/EventLoader.js` — registers `events/*` handlers. `utils/RegisterCommands.js` — pushes slash
   commands to Discord. `utils/CheckIntents.js` runs on `ready`.
5. **Interaction routing**: one `InteractionHandler(interaction, type)` for all four kinds. Four
   `interactionCreate` listeners switch on `isCommand / isButton / isStringSelectMenu / isModalSubmit`
   and call it with the type. `resolveComponent(client[type], customId ?? commandName)` looks up the
   handler; `customId` prefixes route sub-interactions (e.g. `ticket_*`, `embed_*`).

## Permission model
Per-component flags checked in `InteractionHandler`: `admin` (Administrator), `mod` (Manage Server
or Administrator), `owner` (hard-coded user id). Linking/embeds/tickets gate on these.

## Components
- **Commands** (`commands/`): `ping`, `avatar`, `link`, `unlink`, `setlink`, `set`, `create`,
  `embed`, `ticket`.
- **Events** (`events/`): `ready` (~15s poll loop applies pending links — nickname + linked role),
  `GuildJoin`, `GuildLeave`, `MemberJoin`/`MemberLeave` (greet/leave messages), `MemberBoost`.
- **Buttons** (`buttons/`): embed editor (`embed_edit_author/basic/footer/images`), ticket
  (`ticket_close`, `ticket_close_confirm/cancel`).
- **Dropdowns** (`dropdowns/`): `ticket_create` (category select).
- **Modals** (`modals/`): embed editor (`embed_modal_author/basic/footer/images`), ticket forms
  (`ticket_modal_ban/bug/cheater/mute`).

## Key utils
- `Supabase.js` — `client.db` data layer (service key). `NickSync.js` — renames linked members to
  `Name ⛏` + applies/removes the linked role; rate-limit/permission aware (won't touch the owner or
  members above the bot). `TicketManager.js` — opens/closes ticket channels, transcripts.
  `EmbedStore.js` + `embedFormat.js` + `Placeholders.js` — the embed builder + placeholder
  substitution. `EventMessages.js` — greet/leave/boost text. `commandPerms.js`, `resolveComponent.js`,
  `ReadFolder.js`, `CheckIntents.js`.

## Cross-platform features
- **Account linking** → [[shared-supabase]] (the bot reads/writes `link_codes`, `linked_accounts`,
  `link_config`). `/link`, `/unlink`, `/setlink`.
- **Tickets** → `tickets` / `ticket_config` tables; opened via dropdown + modal, managed by
  `TicketManager`. **Embeds** → `embeds` table via the embed builder.

## Related
[[Himawari-System]] · [[shared-supabase]] · [[Himawari-SMP]]
