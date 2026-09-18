import asyncio
import discord
from discord.ext import commands


# =========================
# CONFIGURACIÓN DEL BOT
# =========================

intents = discord.Intents.default()
intents.message_content = True

bot = commands.Bot(
    command_prefix="!",
    intents=intents
)


# =========================
# EVENTO: BOT ENCENDIDO
# =========================

@bot.event
async def on_ready():
    print(f"Bot conectado como {bot.user}")
    print(f"Servidores: {len(bot.guilds)}")


# =========================
# COMANDO: PING
# =========================

@bot.command()
async def ping(ctx):
    await ctx.send("¡Pong!")


# =========================
# COMANDO: AYUDA
# =========================

@bot.command()
async def ayuda(ctx):
    embed = discord.Embed(
        title="Comandos disponibles",
        description="Comandos del bot educativo",
        color=discord.Color.blue()
    )

    embed.add_field(
        name="!ping",
        value="Comprueba si el bot funciona.",
        inline=False
    )

    embed.add_field(
        name="!ayuda",
        value="Muestra esta lista de comandos.",
        inline=False
    )

    embed.add_field(
        name="!bienvenida",
        value="Envía un mensaje de bienvenida.",
        inline=False
    )

    embed.add_field(
        name="!reglas",
        value="Muestra las reglas del servidor.",
        inline=False
    )

    embed.add_field(
        name="!info",
        value="Muestra información del servidor.",
        inline=False
    )

    embed.add_field(
        name="!organizar",
        value="Crea y organiza los canales educativos.",
        inline=False
    )

    embed.add_field(
        name="!canales",
        value="Muestra los canales organizados.",
        inline=False
    )

    embed.add_field(
        name="!limpiar <cantidad>",
        value="Elimina mensajes del canal.",
        inline=False
    )

    await ctx.send(embed=embed)


# =========================
# COMANDO: BIENVENIDA
# =========================

@bot.command()
async def bienvenida(ctx):
    await ctx.send(
        f"¡Bienvenido/a {ctx.author.mention} al servidor educativo! "
        "Aquí encontrarás anuncios, recursos, dudas, tareas y exámenes."
    )


# =========================
# COMANDO: REGLAS
# =========================

@bot.command()
async def reglas(ctx):
    embed = discord.Embed(
        title="Reglas del servidor",
        description="Normas para mantener organizado el servidor educativo.",
        color=discord.Color.green()
    )

    embed.add_field(
        name="1. Respeto",
        value="Trata con respeto a todos los miembros.",
        inline=False
    )

    embed.add_field(
        name="2. No spam",
        value="Evita enviar mensajes repetidos o innecesarios.",
        inline=False
    )

    embed.add_field(
        name="3. Usa los canales correctamente",
        value="Publica cada tema en el canal correspondiente.",
        inline=False
    )

    embed.add_field(
        name="4. Contenido apropiado",
        value="No compartas contenido ofensivo o inapropiado.",
        inline=False
    )

    embed.add_field(
        name="5. Mantén el orden",
        value="Colabora para conservar un ambiente educativo.",
        inline=False
    )

    await ctx.send(embed=embed)


# =========================
# COMANDO: INFORMACIÓN
# =========================

@bot.command()
async def info(ctx):
    guild = ctx.guild

    embed = discord.Embed(
        title="Información del servidor",
        color=discord.Color.blue()
    )

    embed.add_field(
        name="Nombre del servidor",
        value=guild.name,
        inline=False
    )

    embed.add_field(
        name="Cantidad de miembros",
        value=str(guild.member_count),
        inline=True
    )

    embed.add_field(
        name="Cantidad de canales",
        value=str(len(guild.channels)),
        inline=True
    )

    embed.add_field(
        name="Creador del servidor",
        value=str(guild.owner),
        inline=False
    )

    embed.set_footer(
        text="Servidor educativo organizado con un bot de Discord"
    )

    await ctx.send(embed=embed)


# =========================
# COMANDO: ORGANIZAR
# =========================

@bot.command()
@commands.has_permissions(manage_channels=True)
async def organizar(ctx):
    guild = ctx.guild

    await ctx.send("Organizando el servidor...")

    categoria = discord.utils.get(
        guild.categories,
        name="ORGANIZACIÓN"
    )

    if categoria is None:
        categoria = await guild.create_category(
            "ORGANIZACIÓN"
        )
        print("Categoría creada")

    canales = {
        "anuncios": "Noticias y anuncios importantes",
        "recursos-utiles": "Materiales, documentos y recursos",
        "dudas-clase": "Preguntas y dudas sobre las clases",
        "tareas": "Tareas y trabajos",
        "examenes": "Información sobre exámenes",
        "memes": "Memes y contenido divertido"
    }

    creados = 0

    for nombre, tema in canales.items():
        canal = discord.utils.get(
            guild.text_channels,
            name=nombre
        )

        if canal is None:
            await guild.create_text_channel(
                name=nombre,
                category=categoria,
                topic=tema
            )

            creados += 1
            print(f"Canal creado: {nombre}")

        else:
            if canal.category != categoria:
                await canal.edit(category=categoria)

            print(f"Ya existe: {nombre}")

    await ctx.send(
        f"**Servidor organizado correctamente.**\n\n"
        f"Categoría: **ORGANIZACIÓN**\n"
        f"Canales creados: **{creados}**"
    )


# =========================
# COMANDO: VER CANALES
# =========================

@bot.command()
async def canales(ctx):
    guild = ctx.guild

    categoria = discord.utils.get(
        guild.categories,
        name="ORGANIZACIÓN"
    )

    if categoria is None:
        await ctx.send(
            "Todavía no existe la categoría ORGANIZACIÓN."
        )
        return

    if not categoria.channels:
        await ctx.send(
            "La categoría no tiene canales."
        )
        return

    mensaje = "**Canales de organización:**\n\n"

    for canal in categoria.channels:
        mensaje += f"➡️ {canal.mention}\n"

    await ctx.send(mensaje)


# =========================
# COMANDO: LIMPIAR MENSAJES
# =========================

@bot.command()
@commands.has_permissions(manage_messages=True)
async def limpiar(ctx, cantidad: int):
    if cantidad < 1 or cantidad > 100:
        await ctx.send(
            "Debes indicar una cantidad entre 1 y 100."
        )
        return

    await ctx.channel.purge(limit=cantidad + 1)

    mensaje = await ctx.send(
        f"Se eliminaron {cantidad} mensajes."
    )

    await asyncio.sleep(3)
    await mensaje.delete()


# =========================
# MANEJO DE ERRORES
# =========================

@organizar.error
async def organizar_error(ctx, error):
    if isinstance(error, commands.MissingPermissions):
        await ctx.send(
            "No tienes permiso para administrar canales."
        )
    else:
        await ctx.send(
            "Ocurrió un error al organizar el servidor."
        )
        print(error)


@limpiar.error
async def limpiar_error(ctx, error):
    if isinstance(error, commands.MissingPermissions):
        await ctx.send(
            "No tienes permiso para eliminar mensajes."
        )

    elif isinstance(error, commands.MissingRequiredArgument):
        await ctx.send(
            "Debes escribir una cantidad. Ejemplo: !limpiar 10"
        )

    elif isinstance(error, commands.BadArgument):
        await ctx.send(
            "La cantidad debe ser un número. Ejemplo: !limpiar 10"
        )


# =========================
# INICIAR BOT
# =========================

bot.run("TOKEN")
