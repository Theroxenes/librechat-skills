---
name: discord-py-dev
description: >
  Develop Discord bots with discord.py. Use when the user mentions discord.py,
  building a Discord bot in Python, slash commands, prefix commands, cogs,
  intents, embeds, buttons, views, modals, components, event handlers, or any
  discord.py-specific question. Also triggers for "my discord bot isn't working",
  "discord.py error", migrating from 1.x to 2.x, or debugging intents issues.
  Always assume discord.py 2.x unless the user explicitly states 1.x.
---

# discord.py Development (2.x)

## Official References

- **API Reference:** <https://discordpy.readthedocs.io/en/stable/api.html>
- **Guides:** <https://discordpy.readthedocs.io/en/stable/guides/>
- **Source:** <https://github.com/Rapptz/discord.py>

## Version Assumption

Key 1.x → 2.x breaking changes:

| Area | 1.x | 2.x |
|------|-----|-----|
| Slash commands | Not built-in | Native via `discord.app_commands` |
| Components | Not available | Built-in via `discord.ui` |
| `on_message` | Overrides command handler — must call `process_commands` | Same |
| Intents | Optional | Required; enable in code AND Developer Portal |

## Workflow

Ask what they're building, whether they use `commands.Bot` or `discord.Client`, and if they need guild- or global-scoped commands.

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

### 3. Code Rules: async/await everywhere, full import chains, `@bot.tree.command()` for slashes, subclass `discord.ui.View` for components, `ctx.send(embed=embed)` (keyword arg), always call `await bot.process_commands(message)` in `on_message`.

### 4. Intents Checklist

Configure in **both** Discord Developer Portal (Bot → Privileged Gateway Intents) and code:

1. **Discord Developer Portal** → Bot settings → Privileged Gateway Intents
2. **Code:**

```python
import discord

intents = discord.Intents.default()
intents.message_content = True  # required for on_message and prefix commands
# intents.members = True        # only if you need member events/cache
# intents.presence = True       # only if you need presence/status tracking

bot = commands.Bot(command_prefix="!", intents=intents)
```

`PrivilegedIntentsRequired` or `message.content` is `None` → check both places.

## Error Handling

- Per-command: `@command.error()`, global slash: `@bot.tree.error()`, prefix fallback: `on_command_error`. Watch for `MissingPermissions`, `Forbidden`, `CheckFailure`.

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
    await bot.tree.sync()  # Use .guild_sync(guild) for dev

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

1. Intents enabled in code AND Developer Portal?
2. `await bot.process_commands(message)` in `on_message`?
3. Slash commands synced (`bot.tree.sync()`)?
4. Token valid? Bot role permissions correct?
5. Version mismatch (2.x APIs on 1.x install)?
6. Sync/async mixing?

Always cite discord.py docs URLs. Never recommend removed/renamed 1.x APIs without noting the migration path.
