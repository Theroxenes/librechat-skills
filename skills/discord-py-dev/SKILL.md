---
name: discord-py-dev
description: >
  Develop Discord bots with discord.py (the official Python wrapper for the
  Discord API). Use when the user mentions discord.py, building a Discord bot in
  Python, slash commands, prefix commands, cogs, intents, embeds, buttons, views,
  select menus, modals, components, interaction callbacks, event handlers,
  message reactions, permissions, guild management, voice connections, or any
  discord.py-specific question. Also triggers for "my discord bot isn't working",
  "discord.py error", "how to make a Discord bot in Python", migrating from
  discord.py 1.x to 2.x, or debugging intents/privilege gateway issues. Always
  assume discord.py 2.x unless the user explicitly states they are on 1.x.
---

# discord.py Development

Help users build, debug, and ship Discord bots using **discord.py** — the
official Python API wrapper for Discord's gateway and REST APIs.

## Official References

- **API Reference (stable):** <https://discordpy.readthedocs.io/en/stable/api.html>
  - Core client: `discord.Client`, `discord.ext.commands.Bot`
  - App commands (slash): `discord.app_commands`, `app_commands.CommandTree`
  - Components: `discord.ui.View`, `Button`, `Select`, `Modal`
  - Embeds & formatting: `discord.Embed`, `discord.File`, `discord.Attachment`
  - Intents: `discord.Intents`
  - Permissions: `discord.Permissions`, `discord.PermissionOverwrite`
- **Guides:** <https://discordpy.readthedocs.io/en/stable/guides/>
  - Getting started, intents, cogs, error handling, best practices
- **Source:** <https://github.com/Rapptz/discord.py>

## Version Assumption

**Always assume discord.py 2.x** unless the user explicitly states they are on
1.x. Key breaking changes from 1.x → 2.x:

| Area | 1.x | 2.x |
|------|-----|-----|
| Slash commands | Not built-in (required `discord_slash` or forks) | Native via `discord.app_commands` |
| Components (buttons, menus) | Not available | Built-in via `discord.ui` |
| `on_message` overrides command handler | Yes, unless you call `process_commands` | Same — always remind users to add `await bot.process_commands(message)` |
| Intents | Optional for basic bots | Required; must be set explicitly and enabled in Discord Developer Portal |
| `discord.ext.commands` Cog loading | `bot.add_cog()` / `bot.load_extension()` | Same, but slash commands registered via `tree` or cog setup |

## Workflow

### 1. Understand the Goal

Ask:
- What are you trying to build? (command, event handler, interactive component, moderation tool, etc.)
- Are you using `commands.Bot` (prefix + slash hybrid) or bare `discord.Client`?
- Which discord.py version? (default assumption: 2.x)
- Do you need guild-scoped or global commands?

### 2. Identify the API Surface

| Task | Primary API surface |
|------|-------------------|
| Prefix text commands | `discord.ext.commands.Bot`, `@bot.command()`, `@bot.group()` |
| Slash (application) commands | `discord.app_commands`, `@bot.tree.command()`, `@app_commands.describe()` |
| Context menu commands | `@bot.tree.context_menu()`, `AppCommandsContext` |
| Interactive components (buttons, selects, modals) | `discord.ui.View`, `Button`, `Select`, `Modal`, `TextInput` |
| Embeds / rich messages | `discord.Embed`, `Embed.add_field()`, `Embed.set_thumbnail()` |
| Event handling | `@bot.event`, `on_message`, `on_reaction_add`, `on_member_join`, etc. |
| Cogs (modular bots) | `commands.Cog`, `bot.add_cog()`, `bot.load_extension()` |
| Permissions / checks | `commands.has_role()`, `commands.is_owner()`, custom `@commands.check()` |
| Rate limiting / retries | Built-in; discord.py handles Discord rate limits automatically |
| Voice connections | `discord.VoiceClient`, `await channel.connect()` |
| Data persistence | Not built-in — recommend `aiosqlite`, `asyncpg`, or JSON files |

### 3. Produce Correct Code

- Use **async/await** everywhere. discord.py is fully asynchronous.
- Always show the full import chain (e.g., `from discord.ext import commands`).
- For slash commands, use `@bot.tree.command()` on a `commands.Bot` instance, or `@app_commands.command()` on a cog's command tree.
- For components, always subclass `discord.ui.View` and add items with `@child.callback` or `add_item()`.
- When sending messages with embeds, use `await ctx.send(embed=embed)` — never pass embed as a positional arg.
- Remind users that `on_message` **overrides** the built-in command handler; they must call `await bot.process_commands(message)` at the end of their handler or commands stop working.

### 4. Intents Checklist

Every bot needs intents configured in **two places**:

1. **Discord Developer Portal** → Bot settings → Privileged Gateway Intents
   (`message_content`, `members`, `presence`)
2. **Code** — pass an `Intents` object to the bot/client:

```python
import discord

intents = discord.Intents.default()
intents.message_content = True  # required for on_message and prefix commands
# intents.members = True        # only if you need member events/cache
# intents.presence = True       # only if you need presence/status tracking

bot = commands.Bot(command_prefix="!", intents=intents)
```

If the user gets `PrivilegedIntentsRequired` or their events fire but
`message.content` is `None`, this is almost always the cause.

### 5. Error Handling

- Use `@command.error()` for per-command error handling.
- Use `@bot.tree.error()` for global slash command errors.
- Use `@bot.event async def on_command_error(ctx, error)` for prefix command fallbacks.
- Common errors to anticipate:
  - `commands.MissingPermissions` — bot lacks a permission the user expects
  - `discord.Forbidden` — bot role is below target role, or missing channel perms
  - `app_commands.CheckFailure` — custom check failed on slash command

## Common Patterns

### Minimal Prefix Bot

```python
import discord
from discord.ext import commands

intents = discord.Intents.default()
intents.message_content = True

bot = commands.Bot(command_prefix="!", intents=intents)

@bot.event
async def on_ready():
    print(f"Logged in as {bot.user} (ID: {bot.user.id})")

@bot.command()
async def ping(ctx):
    await ctx.send("Pong!")

bot.run("YOUR_TOKEN")
```

### Minimal Slash Command Bot

```python
import discord
from discord.ext import commands

intents = discord.Intents.default()
intents.message_content = True

bot = commands.Bot(command_prefix="!", intents=intents)

@bot.event
async def on_ready():
    print(f"Logged in as {bot.user}")
    # Sync slash commands globally (or use guild sync for dev):
    # await bot.tree.sync()

@bot.tree.command(name="ping", description="Ping the bot")
async def ping(interaction: discord.Interaction):
    await interaction.response.send_message("Pong!", ephemeral=False)

bot.run("YOUR_TOKEN")
```

### Button / View Component

```python
import discord
from discord.ui import View, Button

class MyView(View):
    @discord.ui.button(label="Click Me", style=discord.ButtonStyle.primary)
    async def click(self, interaction: discord.Interaction, button: Button):
        await interaction.response.send_message("You clicked the button!", ephemeral=True)

# Usage in a command:
# await ctx.send("Here is a button:", view=MyView())
```

### Embed Message

```python
embed = discord.Embed(
    title="Example Embed",
    description="This is a rich embed message.",
    color=discord.Color.blue(),
)
embed.add_field(name="Field 1", value="Value 1", inline=True)
embed.set_footer(text="Powered by discord.py")
await ctx.send(embed=embed)
```

### Cog Structure

```python
import discord
from discord.ext import commands

class MyCog(commands.Cog):
    def __init__(self, bot: commands.Bot):
        self.bot = bot

    @commands.command()
    async def cog_command(self, ctx):
        await ctx.send("This command lives in a cog!")

    @discord.app_commands.command(name="slash_in_cog", description="A slash command in a cog")
    async def slash_in_cog(self, interaction: discord.Interaction):
        await interaction.response.send_message("Slash from a cog!", ephemeral=True)

async def setup(bot: commands.Bot):
    await bot.add_cog(MyCog(bot))
```

## Debugging Checklist

When the user reports an issue:

1. **Intents** — are the right intents enabled in code AND Developer Portal?
2. **`on_message` override** — did they forget `await bot.process_commands(message)`?
3. **Command sync** — slash commands need `await bot.tree.sync()` after changes during development.
4. **Token** — is the bot token correct and not expired/revoked?
5. **Permissions** — does the bot role have Manage Messages, Send Messages, etc.?
6. **Version mismatch** — are they using 2.x APIs on a 1.x install (or vice versa)?
7. **Event loop** — are they mixing sync and async code incorrectly?

## Tone

Be direct and technical. Show runnable code snippets. Always cite the relevant
discord.py docs URL so the user can read further. Never recommend an API that
was removed or renamed in 2.x without noting the migration path.
