import re
import datetime
from datetime import datetime, timedelta
import asyncio
import discord
from discord.ext import commands

intents = discord.Intents.default()
intents.typing = False
intents.presences = False
intents.message_content = True

bot = commands.Bot(command_prefix='/', intents=intents)

bot.remove_command('help')

guilds = {}  # Dictionary to store guild information


@bot.event
async def on_ready():
    print(f'Bot is ready. Logged in as {bot.user.name} ({bot.user.id})')
    
    for guild in bot.guilds:
        guilds[guild.id] = {
            "name": guild.name,
            "influence": 0,
            "color": discord.Color.default()
        }
@bot.command()
async def ping(ctx):
    latency = bot.latency
    await ctx.send(f"Pong! Latency: {latency * 1000}ms")

@bot.command()
async def dhelp(ctx):
    command_list = bot.commands
    response = "List of available commands:\n\n"
    for command in command_list:
        response += f"Command: {command.name}\n"
        response += f"Description: {command.help}\n\n"
    await ctx.send(response)

@bot.command()
async def identifyuser(ctx, target_user: discord.Member = None):
    if target_user is None:
        await ctx.send("Please mention a valid user.")
        return

    try:
        guild_info = guilds.get(ctx.guild.id)
        guild_name_role = discord.utils.get(target_user.roles, name=f"{ctx.guild.name} (dugs)")


        bot_created_roles = [role for role in target_user.roles if role.name.endswith("(dugs)")]
        regular_roles = [role for role in target_user.roles if role.name != '@everyone' and not role.name.endswith('(dugs)')]

        if bot_created_roles:
            response = f"Target username: {target_user.name}\n"
            response += f"Target ID: {target_user.id}\n"
            response += f"Target discriminator: {target_user.discriminator}\n"
            response += f"Target joined at: {target_user.joined_at}\n\n"

            guild_info_text = f"Guild Info:\n"
            if guild_name_role is not None:
                guild_info_text += f"Guild Name: {guild_name_role.name}\n"
                guild_info_text += f"Member Count: {guild_name_role.guild.member_count}\n\n"

            bot_created_roles_text = f"Guild:\n"
            for role in bot_created_roles:
                bot_created_roles_text += f"- {role.mention}\n"

            await ctx.send(response)
            await ctx.send(guild_info_text)
            await ctx.send(bot_created_roles_text)

        if regular_roles:
            roles_text = f"Roles:\n"
            for role in regular_roles:
                roles_text += f"- {role.mention}\n"

            await ctx.send(roles_text)

    except discord.NotFound:
        error_message = "User not found in the server."
        await ctx.send(error_message)
        print(error_message)


    except discord.NotFound:
        error_message = "User not found in the server."
        await ctx.send(error_message)
        print(error_message)

@bot.command()
async def createguild(ctx, guild_name: str, guild_type: str):
    existing_guild_name_role = discord.utils.get(ctx.guild.roles, name=guild_name + " (dugs)")
    guild_leader_role = discord.utils.get(ctx.guild.roles, name="Guild Leader")

    if existing_guild_name_role:
        await ctx.send("A guild with that name already exists.")
        return

    if guild_leader_role in ctx.author.roles:
        await ctx.send("You are already a Guild Leader.")
        return

    try:
        guild_name_role = await ctx.guild.create_role(name=guild_name + " (dugs)")

        if guild_type.lower() == "public":
            await ctx.author.add_roles(guild_leader_role, guild_name_role)
            await ctx.send(f"Guild '{guild_name}' created. You are now the Guild Leader.")

            # Allow other members to join the guild
            await ctx.send(f"Other members can join the guild by using the command: /joinguild {guild_name}")
        elif guild_type.lower() == "private":
            await ctx.author.add_roles(guild_leader_role, guild_name_role)
            await ctx.send(f"Guild '{guild_name}' created. You are now the Guild Leader. It is a private guild.")

    except discord.Forbidden:
        await ctx.send("I don't have the necessary permissions to create roles.")


@bot.command()
async def leaveguild(ctx):
    bot_created_roles = [role for role in ctx.author.roles if role.name.endswith("(dugs)")]
    guild_leader_role = discord.utils.get(ctx.guild.roles, name="Guild Leader")

    if not bot_created_roles:
        await ctx.send("You are not a member of any guild.")
        return

    try:
        for role in bot_created_roles:
            await ctx.author.remove_roles(role)

        # Remove guild leader role
        if guild_leader_role in ctx.author.roles:
            await ctx.author.remove_roles(guild_leader_role)

        await ctx.send("You have left the guild.")

    except discord.Forbidden:
        await ctx.send("I don't have the necessary permissions to remove roles.")


@bot.command()
async def joinguild(ctx, guild_name: str):
    guild_name_role = discord.utils.get(ctx.guild.roles, name=guild_name + " (dugs)")

    if guild_name_role:
        if guild_name_role.permissions.administrator:
            await ctx.author.add_roles(guild_name_role)
            await ctx.send(f"You have joined the guild '{guild_name}'.")
        else:
            await ctx.send(f"The guild '{guild_name}' is private. You need an invitation to join.")
    else:
        await ctx.send(f"The guild '{guild_name}' does not exist.")

@bot.command()
async def invitetoguild(ctx, user: discord.Member):
    guild_roles = [role for role in ctx.author.roles if role.name.endswith("(dugs)")]

    if not guild_roles:
        await ctx.send("You are not a member of any guild.")
        return

    try:
        invite_message = await ctx.send(f"{user.mention}, you have been invited to join the guild '{guild_roles[0].name}'. "
                                        "React with ✅ to accept the invitation or ❌ to decline.")
        await invite_message.add_reaction("✅")
        await invite_message.add_reaction("❌")

        def check(reaction, invited_user):
            return invited_user == user and reaction.message.id == invite_message.id and str(reaction.emoji) in ["✅", "❌"]

        reaction, _ = await bot.wait_for("reaction_add", check=check)

        if str(reaction.emoji) == "✅":
            guild_role = guild_roles[0]
            await user.add_roles(guild_role)
            await ctx.send(f"{user.mention} has accepted the invitation and joined the guild '{guild_role.name}'.")
        else:
            await ctx.send(f"{user.mention} has declined the invitation to join the guild '{guild_roles[0].name}'.")
    except discord.Forbidden:
        await ctx.send("I don't have the necessary permissions to add roles or react to the message.")



@bot.command()
@commands.has_role("Guild Leader")
async def resignfromleader(ctx):
    guild_leader_role = discord.utils.get(ctx.guild.roles, name="Guild Leader")

    if guild_leader_role not in ctx.author.roles:
        await ctx.send("You are not a Guild Leader.")
        return

    try:
        await ctx.author.remove_roles(guild_leader_role)
        await ctx.send("You have resigned from the Guild Leader role.")

    except discord.Forbidden:
        await ctx.send("I don't have the necessary permissions to remove roles.")


@bot.command()
@commands.has_permissions(administrator=True)
@commands.is_owner()
async def clearallguilds(ctx):
    if ctx.author.id != ctx.guild.owner_id:
        # Check if the command author is not the server owner
        await ctx.send("Only the server owner or someone with administrator privileges can use this command.")
        await ctx.message.delete()  # Delete the command message
        return
    
    for guild in bot.guilds:
        try:
            guild_leader_role = discord.utils.get(guild.roles, name="Guild Leader")

            guild_leader_members = [member for member in guild.members if guild_leader_role in member.roles]

            for member in guild_leader_members:
                await member.remove_roles(guild_leader_role)

            bot_created_roles = [role for role in guild.roles if role.name.endswith("(dugs)")]

            for role in bot_created_roles:
                await role.delete()

            await guild.delete()

            await ctx.send(f"Guild {guild.name} has been cleared.")

        except discord.Forbidden:
            await ctx.send(f"Insufficient permissions to clear guild: {guild.name}")

    await ctx.send("All guilds have been cleared.")
    await ctx.message.delete()  # Delete the command message

@bot.command()
async def showallguilds(ctx):
    bot_created_roles = [role for role in ctx.guild.roles if role.name.endswith("(dugs)")]

    if bot_created_roles:
        guild_info_list = []
        for role in bot_created_roles:
            guild_name = role.name[:-6]  # Remove the "(dugs)" suffix
            guild = discord.utils.get(ctx.guild.roles, name=guild_name)

            guild_leader = None
            member_count = 0

            if guild:
                guild_members = guild.members
                guild_leader = discord.utils.get(guild_members, roles=role)
                member_count = len(guild_members)

            guild_info = f"Guild Name: {guild_name}\n"
            guild_info += f"Guild Role ID: {role.id}\n"
            guild_info += f"Guild Role Color: {role.color}\n"
            guild_info += f"Guild Leader: {guild_leader.name if guild_leader else 'Not Found'}\n"
            guild_info += f"Member Count: {member_count}\n\n"

            guild_info_list.append(guild_info)

        guild_info_text = "\n".join(guild_info_list)
        await ctx.send(f"List of Guilds:\n{guild_info_text}")
    else:
        await ctx.send("No guilds found.")

@bot.command()
async def memberlist(ctx, guild_name: str):
    guild_roles = [role for role in ctx.author.roles if role.name.endswith("(dugs)")]

    if not guild_roles:
        await ctx.send("You are not a member of any guild.")
        return

    guild_name_role = discord.utils.get(ctx.guild.roles, name=guild_name + " (dugs)")

    if guild_name_role not in guild_roles:
        await ctx.send(f"You are not a member of the guild '{guild_name}'.")
        return

    guild = discord.utils.get(bot.guilds, name=guild_name)

    if guild:
        members_with_role = [member for member in guild.members if guild_name_role in member.roles]

        if members_with_role:
            response = f"Member list for {guild_name}:\n\n"
            for member in members_with_role:
                response += f"Member: {member.name}#{member.discriminator}\n"
                response += f"Joined at: {member.joined_at}\n\n"
            await ctx.send(response)
        else:
            await ctx.send(f"No members found with the role '{guild_name}'.")
    else:
        await ctx.send(f"A guild with the name '{guild_name}' was not found.")

def get_guild_role(guild, role_name):
    role_name = role_name.lower()
    for role in guild.roles:
        if role.name.lower() == role_name:
            return role
    return None

@bot.command()
async def declarewar(ctx, duration: str, target_guild: discord.Role, target_user: discord.Member):

    duration_match = re.match(r"(\d+)\s*(\w+)", duration)
    if not duration_match:
        await ctx.send("Invalid duration format. Please specify the duration as <value> <unit> (e.g., 3 hours).")
        return

    duration_value = int(duration_match.group(1))
    duration_unit = duration_match.group(2).lower()

    if duration_unit == "minute" or duration_unit == "minutes":
        duration_seconds = duration_value * 60
    elif duration_unit == "hour" or duration_unit == "hours":
        duration_seconds = duration_value * 3600
    elif duration_unit == "day" or duration_unit == "days":
        duration_seconds = duration_value * 86400
    else:
        await ctx.send("Invalid duration unit. Please specify the duration as minutes, hours, or days.")
        return

    end_time = datetime.utcnow() + timedelta(seconds=duration_seconds)

    prompt_message = await ctx.send(f"War declared against '{target_guild.name}' for {duration}!\n"
                                    f"{target_user.mention}, react with ✅ to accept the war or ❌ to decline.")

    await prompt_message.add_reaction("✅")
    await prompt_message.add_reaction("❌")

    def check(reaction, user):
        return user == target_user and reaction.message.id == prompt_message.id and str(reaction.emoji) in ["✅", "❌"]

    try:
        reaction, _ = await bot.wait_for("reaction_add", timeout=duration_seconds, check=check)

        if str(reaction.emoji) == "✅":

            await ctx.send(f"The war against '{target_guild.name}' has been accepted by '{target_user.name}'!")


            await asyncio.sleep((end_time - datetime.utcnow()).total_seconds())

            team_influence = {
                "Team A": 0,
                "Team B": 0
            }

            async for message in ctx.channel.history(limit=None):
                if not message.author.bot and message.guild.id in guilds:
                    influence = guilds[message.guild.id].get("influence", 0)

                    words = message.content.split()

                    valid_words = [word for word in words if len(word) >= 2]

                    if len(valid_words) >= 3:  # Check if there are at least 3 valid words
                        influence += 1  # Add 1 influence for a regular message

                    elif message.attachments:
                        for attachment in message.attachments:
                            if attachment.filename.lower().endswith(('.jpg', '.jpeg', '.png', '.gif')):
                                influence += 5  # Add 5 influence for an image attachment
                            elif attachment.filename.lower().endswith(('.mp4', '.mov', '.avi')):
                                influence += 10  # Add 10 influence for a video attachment

                    guilds[message.guild.id]["influence"] = influence
                    team_influence[message.guild.name] += influence


            winning_team = max(team_influence, key=team_influence.get)
            winning_influence = team_influence[winning_team]


            await ctx.send(f"The war against '{target_guild.name}' has ended!")
            await ctx.send("Influence changes during the war:")

            for team, influence in team_influence.items():
                await ctx.send(f"{team}: {influence}")

            await ctx.send(f"The winning team is '{winning_team}' with {winning_influence} influence points!")

        else:

            await ctx.send(f"The war against '{target_guild.name}' has been declined by '{target_user.name}'!")

    except asyncio.TimeoutError:
        # War request timed out
        await ctx.send(f"The war request against '{target_guild.name}' has timed out.")
@bot.command()
async def influence(ctx):
    guild_roles = [role for role in ctx.author.roles if role.name.endswith("(dugs)")]

    if not guild_roles:
        await ctx.send("You are not a member of any guild.")
        return

    guild_name_role = guild_roles[0]

    guild_info = guilds.get(ctx.guild.id)
    if guild_info is None:
        guild_info = {"influence": 0}
        guilds[ctx.guild.id] = guild_info

    current_influence = guild_info["influence"]

    await ctx.send(f"The guild '{guild_name_role.name}' has {current_influence} influence point(s).")

team_influence = {}  # Define an empty dictionary

@bot.event
async def on_message(message):
    if not message.author.bot:
        guild_id = message.guild.id

        if guild_id not in team_influence:
            team_influence[guild_id] = 0

        influence = team_influence[guild_id]


        words = message.content.split()


        valid_words = [word for word in words if len(word) >= 2]

        if len(valid_words) >= 3:  # Check if there are at least 3 valid words
            influence += 1  # Add 1 influence for a regular message

        elif message.attachments:
            for attachment in message.attachments:
                if attachment.filename.lower().endswith(('.jpg', '.jpeg', '.png', '.gif')):
                    influence += 5  # Add 5 influence for an image attachment
                elif attachment.filename.lower().endswith(('.mp4', '.mov', '.avi')):
                    influence += 10  # Add 10 influence for a video attachment

        team_influence[guild_id] = influence

    await bot.process_commands(message)

@bot.command()
@commands.has_role("Guild Leader")
async def changebannercolor(ctx, color: discord.Color):
    guild_info = guilds.get(ctx.guild.id)
    if guild_info is None:
        await ctx.send("Guild information not found.")
        return

    guild_role = discord.utils.get(ctx.guild.roles, name=guild_info["name"] + " (dugs)")
    if guild_role is None:
        await ctx.send("Guild role not found.")
        return

    try:
        await guild_role.edit(color=color)
        guild_info["color"] = color

        members_with_role = [member for member in ctx.guild.members if guild_role in member.roles]
        for member in members_with_role:
            await member.edit(color=color)

        await ctx.send("Guild banner color has been changed.")

    except discord.Forbidden:
        await ctx.send("I don't have the necessary permissions to edit roles.")



bot.run('Your  bot token')
