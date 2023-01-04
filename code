import requests
import telegram
from telegram.ext import Updater, MessageHandler, Filters, CallbackQueryHandler, CommandHandler
import sqlite3
import os

# Replace YOUR_TELEGRAM_BOT_TOKEN with your actual bot token
updater = Updater(token='YOUR_TELEGRAM_BOT_TOKEN', use_context=True)
headers = {'Authorization': 'Bearer YOUR_DALL-E_API_KEY'}
dispatcher = updater.dispatcher
last_message = None

repeat_button = telegram.InlineKeyboardButton('Повторити', callback_data='repeat')
keyboard = [[repeat_button]]
reply_markup = telegram.InlineKeyboardMarkup(keyboard)

# File paths of the pictures to send
picture1_path = '1.jpeg'
picture2_path = '2.jpeg'
picture3_path = '3.jpeg'
picture4_path = '4.jpeg'

# Descriptions of the pictures
picture1_description = 'Magic book'
picture2_description = 'Elephant Children book illustration'
picture3_description = 'Vintage, grainy, sepia, photograph of adorned Native American man, Hassleblad, f4, 150mm, ' \
                      'natural light '
picture4_description = 'Hyperrealistic pencil drawing'


def handle_message(update, context):
   # Connect to the database
   conn = sqlite3.connect('users.db')

   # Create a cursor
   cursor = conn.cursor()

   # Create the "users" table if it doesn't exist
   cursor.execute("CREATE TABLE IF NOT EXISTS users (telegram_id INTEGER PRIMARY KEY, count INTEGER)")

   # Extract the text of the message and use it as the input for the DALL-E API
   if update.message.text == 'stat':
       # Check if the user is authorized to use the command
       if update.message.from_user.id == myID:
           # Connect to the database
           conn = sqlite3.connect('users.db')
           cursor = conn.cursor()

           # Query the database for the number of users
           cursor.execute("SELECT COUNT(*) FROM users")
           result = cursor.fetchone()
           count = result[0]

           # Send the result back to the user
           context.bot.send_message(chat_id=update.effective_chat.id, text=f'There have been {count} users of the bot.')

   global last_message
   last_message = update.message.text
   text = update.message.text

   data = {'prompt': text, 'model': 'image-alpha-001', 'num_images': 1}
   response = requests.post('https://api.openai.com/v1/images/generations', headers=headers, json=data)
   response_json = response.json()

   if 'data' in response_json:
       # Extract the generated image URL from the response
       image_url = response_json['data'][0]['url']

       # Use the Telegram API to send the generated image back to the user
       context.bot.send_photo(chat_id=update.effective_chat.id, photo=image_url)
   else:
       # Send an error message to the user
       context.bot.send_message(chat_id=update.effective_chat.id, text='Вибачай, але DALL-E не прислав ніякого '
                                                                       'зображення. Спробуй ще раз з інакшим описом.')

   # Send the "Start Again" button to the user
   context.bot.send_message(chat_id=update.effective_chat.id, text='Хочеш сгенерувати нове зображення? Чи повторити '
                                                                   'свій пошук?', reply_markup=reply_markup)

   # Update the user data in the database
   user_id = update.message.from_user.id
   cursor.execute("SELECT count FROM users WHERE telegram_id=?", (user_id,))
   result = cursor.fetchone()
   if result:
       # Increment the counter for the user
       count = result[0] + 1
       cursor.execute("UPDATE users SET count=? WHERE telegram_id=?", (count, user_id))
       conn.commit()
   else:
       # Insert a new row for the user and set the counter to 1
       cursor.execute("INSERT INTO users (telegram_id, count) VALUES (?, 1)", (user_id,))
       conn.commit()

   # Close the connection
   conn.close()


def start(update, context):
   # Send a welcome message to the user
   context.bot.send_message(chat_id=update.effective_chat.id, text='Тебе вітає бот DALL-E, який генерує зображення '
                                                                   'по опису! Просто напиши мені що ти хочешь '
                                                                   'побачити та очікуй на результат!')


def repeat(update, context):
   # Extract the text of the last message and use it as the input for the DALL-E API
   global last_message
   text = last_message
   data = {'prompt': text, 'model': 'image-alpha-001', 'num_images': 1}
   response = requests.post('https://api.openai.com/v1/images/generations', headers=headers, json=data)
   response_json = response.json()

   if 'data' in response_json:
       # Extract the generated image URL from the response
       image_url = response_json['data'][0]['url']

       # Use the Telegram API to send the generated image back to the user
       context.bot.send_photo(chat_id=update.effective_chat.id, photo=image_url)
       context.bot.send_message(chat_id=update.effective_chat.id, text='Would you like to generate another?',
                                reply_markup=reply_markup)
   else:
       # Send an error message to the user
       context.bot.send_message(chat_id=update.effective_chat.id, text='Вибач, я загубив твій останній опис.')
       context.bot.send_message(chat_id=update.effective_chat.id, text='Would you like to generate another?')


def command1(update, context):
   # Send picture 1 with its description
   context.bot.send_photo(chat_id=update.effective_chat.id, photo=open(picture1_path, 'rb'))
   context.bot.send_message(chat_id=update.effective_chat.id, text=picture1_description)

   # Send picture 2 with its description
   context.bot.send_photo(chat_id=update.effective_chat.id, photo=open(picture2_path, 'rb'))
   context.bot.send_message(chat_id=update.effective_chat.id, text=picture2_description)

   # Send picture 3 with its description
   context.bot.send_photo(chat_id=update.effective_chat.id, photo=open(picture3_path, 'rb'))
   context.bot.send_message(chat_id=update.effective_chat.id, text=picture3_description)

   # Send picture 4 with its description
   context.bot.send_photo(chat_id=update.effective_chat.id, photo=open(picture4_path, 'rb'))
   context.bot.send_message(chat_id=update.effective_chat.id, text=picture4_description)


command1_handler = CommandHandler('command1', command1)
dispatcher.add_handler(command1_handler)

# Set up a message handler function to process incoming messages
message_handler = MessageHandler(Filters.text, handle_message)
dispatcher.add_handler(message_handler)

repeat_callback_handler = CallbackQueryHandler(repeat, pattern='repeat')
dispatcher.add_handler(repeat_callback_handler)

# Start the bot
updater.start_polling()
