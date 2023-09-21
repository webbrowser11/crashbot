# crashbot
a discord bot

code:

```
import json
import os
import random


import discord
from discord.ext import commands
os.system("python flask.py")

intents = discord.Intents.default()
intents.message_content = True

bot = commands.Bot(command_prefix='!', intents=intents)

# Define the server (guild) ID
bot_command_server_id = 1150804828268798042  # Your server (guild) ID here



# Initialize user levels and XP from 'levels.json' if it exists
user_levels = {}

try:
    if os.path.exists('levels.json'):
        with open('levels.json', 'r') as file:
            try:
                user_levels = json.load(file)
            except json.JSONDecodeError as e:
                print(f"Error decoding 'levels.json': {e}")
                user_levels = {}  # Create an empty dictionary in case of an error
    else:
        user_levels = {}

except Exception as e:
    print(f"Error while reading 'levels.json': {e}")

@bot.event
async def on_ready():
    print(f'We have logged in as {bot.user}')

@bot.command(guild_ids=[bot_command_server_id])  # Specify the server's ID here
async def ping(ctx):
    # Calculate the latency (ping) of the bot
    latency = round(bot.latency * 1000)  # Convert to milliseconds

    # Create an embedded message to display the ping
    embed = discord.Embed(
        title="Ping",
        color=0x00ff00,  # Green color
        description=f"Latency: {latency}ms"
    )

    await ctx.send(embed=embed)


  
@bot.command(guild_ids=[bot_command_server_id])  # Specify the server's ID here
async def level(ctx):
    author_id = str(ctx.author.id)

    # Check if the user is in the user_levels dictionary
    if author_id in user_levels:
        user_data = user_levels[author_id]
        level = user_data['level']
        xp = user_data['xp']
        xp_required = user_data['xp_required']  # Add this line to retrieve required XP





            # Create an embedded message to display the user's level, XP, and avatar
        embed = discord.Embed(
                title="Level",
                color=0x818589,  # Gray color
                description=f"Level {level}\nXP: {xp}/{xp_required}",)

              

        await ctx.send(embed=embed)
    else:
        await ctx.send("Unable to retrieve user information.")
    






@bot.event
async def on_message(message):
    if message.author == bot.user:
        return

    # Check if the message is in the 'general' channel
    if message.channel.name == 'general':
        author_id = str(message.author.id)

        # Check if the user is in the user_levels dictionary
        if author_id not in user_levels:
            user_levels[author_id] = {
                'level': 0,
                'xp': 0,
                'xp_required': 100  # Initial XP required to level up
            }

        # Calculate XP gained (between 7 and 10)
        xp_gained = random.randint(7, 10)
        user_levels[author_id]['xp'] += xp_gained

        while user_levels[author_id]['xp'] >= user_levels[author_id]['xp_required']:
            user_levels[author_id]['xp'] -= user_levels[author_id]['xp_required']
            user_levels[author_id]['level'] += 1
            user_levels[author_id]['xp_required'] += 15  # Increase required XP by 15

            # Send a message in the 'general' channel when the user levels up
            general_channel = discord.utils.get(message.guild.text_channels, name='general')
            if general_channel:
                await general_channel.send(f'{message.author.mention} has leveled up to level {user_levels[author_id]["level"]}!')

        # Save updated user levels and XP to 'levels.json'
        try:
            with open('levels.json', 'w') as file:
                json.dump(user_levels, file)
        except Exception as e:
            print(f"Error while writing 'levels.json': {e}")

    await bot.process_commands(message)

try:
    token = os.getenv("TOKEN") or ""
    if token == "":
        raise Exception("Please add your bot token to the Secrets pane.")
    bot.run(token)
except discord.HTTPException as e:
    if e.status == 429:
        print("The Discord servers denied the connection for making too many requests")
        print("Get help from https://stackoverflow.com/questions/66724687/in-discord-py-how-to-solve-the-error-for-toomanyrequests")
    else:
        raise e
except Exception as e:
    print(f"An error occurred while running the bot: {e}")


```
