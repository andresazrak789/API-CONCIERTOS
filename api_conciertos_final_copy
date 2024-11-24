import telebot
import requests
from telebot import types
import google.generativeai as genai
import os

# Configuración de las claves API
os.environ["API_KEY"] = "AIzaSyC1uhmOt89ntEY7trNeAUbTAtNkwcMXbJg"  # Reemplaza con tu API Key de Google
genai.configure(api_key=os.environ["API_KEY"])

# Token del bot de Telegram
TOKEN = '7571232903:AAF6_CnQviBkr4GWjMcDKxE0w6qa9RvajGA'
bot = telebot.TeleBot(TOKEN)

# Configuración de Spotify
client_id = '8fe819ee033e4a6a86ebac251c4245c4'
client_secret = '1f53f4f2543b459a90869e3a1cae633a'

# Función para obtener el token de acceso de Spotify
def get_access_token(client_id, client_secret):
    token_url = "https://accounts.spotify.com/api/token"
    headers = {"Content-Type": "application/x-www-form-urlencoded"}
    data = {
        "grant_type": "client_credentials",
        "client_id": client_id,
        "client_secret": client_secret
    }
    response = requests.post(token_url, headers=headers, data=data)
    if response.status_code == 200:
        return response.json()['access_token']
    else:
        raise Exception(f"Error al obtener el token: {response.status_code} - {response.text}")

# Función para obtener el ID de un artista en Spotify
def get_artist_id(access_token, artist_name):
    search_url = f"https://api.spotify.com/v1/search?q={artist_name}&type=artist"
    headers = {"Authorization": f"Bearer {access_token}"}
    response = requests.get(search_url, headers=headers)
    if response.status_code == 200:
        artists = response.json().get('artists', {}).get('items', [])
        if artists:
            return artists[0]['id']
        else:
            raise Exception("Artista no encontrado.")
    else:
        raise Exception(f"Error al buscar el artista: {response.status_code} - {response.text}")

# Función para obtener información relacionada de un artista desde Spotify
def get_artist_info(access_token, artist_name):
    artist_id = get_artist_id(access_token, artist_name)
    url = f"https://api.spotify.com/v1/artists/{artist_id}"
    headers = {"Authorization": f"Bearer {access_token}"}
    response = requests.get(url, headers=headers)
    if response.status_code == 200:
        return response.json()
    else:
        raise Exception(f"Error al obtener información del artista: {response.status_code} - {response.text}")

# Función para obtener artistas relacionados
def get_related_artists(access_token, artist_name):
    artist_id = get_artist_id(access_token, artist_name)
    url = f"https://api.spotify.com/v1/artists/{artist_id}/related-artists"
    headers = {"Authorization": f"Bearer {access_token}"}
    response = requests.get(url, headers=headers)
    if response.status_code == 200:
        return response.json()
    else:
        raise Exception(f"Error al obtener artistas relacionados: {response.status_code} - {response.text}")

# Función para obtener merchandising de un artista desde Mercado Libre (simulación)
def get_merch_from_mercadolibre(artist_name):
    # Se hace una búsqueda básica en Mercado Libre por el nombre del artista
    # Este URL es un ejemplo. Dependiendo de cómo quieras buscar, ajusta la URL
    search_url = f"https://listado.mercadolibre.com.ar/{artist_name.replace(' ', '-').lower()}-merchandising"
    return search_url

# Función para mostrar la información del artista
def show_artist_info(message, artist_name):
    try:
        token = get_access_token(client_id, client_secret)
        artist_info = get_artist_info(token, artist_name)

        # Información básica del artista
        artist_name = artist_info['name']
        artist_genres = ', '.join(artist_info.get('genres', [])) or "Géneros no disponibles"
        artist_popularity = artist_info.get('popularity', 'N/A')
        artist_followers = artist_info.get('followers', {}).get('total', 'N/A')

        # Componer el mensaje con información básica
        artist_message = (
            f"**Información sobre {artist_name}:**\n\n"
            f"**Géneros:** {artist_genres}\n"
            f"**Popularidad:** {artist_popularity}\n"
            f"**Seguidores:** {artist_followers}\n\n"
        )

        # Utilizar el modelo de IA para obtener más detalles sobre el artista
        pregunta = f"detallame muy breve y ordenada, información del artista, su estilo y estetica {artist_name}."
        respuesta = genai.GenerativeModel('gemini-1.5-flash-latest').generate_content(pregunta)

        if respuesta and hasattr(respuesta, 'text'):
            artist_message += f"**Descripción:** {respuesta.text}"

        # Enviar la información del artista al usuario
        bot.send_message(message.chat.id, artist_message)

        # Obtener el merchandising del artista
        merch_url = get_merch_from_mercadolibre(artist_name)
        bot.send_message(message.chat.id, f"¡Aquí está el merchandising disponible para {artist_name} en Mercado Libre! \n{merch_url}")

    except Exception as e:
        bot.send_message(message.chat.id, f"Error al obtener información del artista: {e}")

# Función para mostrar artistas relacionados con botones para Mercado Libre
def show_related_artists(message, artist_name):
    try:
        token = get_access_token(client_id, client_secret)
        related_artists = get_related_artists(token, artist_name)

        if not related_artists['artists']:
            bot.send_message(message.chat.id, "No se encontraron artistas relacionados.")
            return

        # Limitar la lista de artistas a los primeros 5
        related_artists_list = related_artists['artists'][:5]

        # Crear los botones para cada artista relacionado con enlaces a Mercado Libre
        markup = types.InlineKeyboardMarkup()
        for artist in related_artists_list:
            artist_name_clean = artist['name'].replace(' ', '-').lower()
            mercado_libre_url = f"https://listado.mercadolibre.com.ar/{artist_name_clean}-merchandising"
            button = types.InlineKeyboardButton(artist["name"], url=mercado_libre_url)
            markup.add(button)

        # Enviar el mensaje con los botones
        bot.send_message(message.chat.id, f"Como vimos que te gusto mucho el merch de {artist_name}, a continuación te recomendamos el de otros artistas para que puedas inspirarte", reply_markup=markup)

        # Despedida final, mensaje con el mensaje que solicitaste
        bot.send_message(message.chat.id, "Ya estás listo/a para tu show!")

    except Exception as e:
        bot.send_message(message.chat.id, f"Error al obtener artistas relacionados: {e}")

# Comando para obtener información del artista
@bot.message_handler(commands=['artista'])
def handle_artist(message):
    try:
        artist_name = message.text.split(maxsplit=1)[1]
        show_artist_info(message, artist_name)
        show_related_artists(message, artist_name)  # Llamamos a la función para mostrar artistas relacionados
    except IndexError:
        bot.send_message(message.chat.id, "Por favor, proporciona el nombre de un artista después del comando /artista.")
    except Exception as e:
        bot.send_message(message.chat.id, f"Error al procesar el comando: {e}")

# Comando de bienvenida
@bot.message_handler(commands=['start'])
def send_welcome(message):
    bot.send_message(message.chat.id, "¡Bienvenido/a! Soy Wanda de Spotmel y te ayudo a que encuentres merch para tu próximo show. Usa /artista <nombre> para obtener información.")

# Responder a cualquier mensaje con la IA
@bot.message_handler(func=lambda message: True)
def responder_mensaje(message):
    pregunta = message.text
    try:
        respuesta = genai.GenerativeModel('gemini-1.5-flash-latest').generate_content(pregunta)
        if respuesta and hasattr(respuesta, 'text'):
            bot.send_message(message.chat.id, respuesta.text)
        else:
            bot.send_message(message.chat.id, "No pude generar una respuesta. Intenta de nuevo más tarde.")
    except Exception as e:
        bot.send_message(message.chat.id, f"Error al generar la respuesta: {e}")

print("Bot de Telegram iniciado")
bot.polling()
