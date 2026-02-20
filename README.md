import telebot
from telebot import types
import os

TOKEN = os.getenv("TOKEN")
bot = telebot.TeleBot(TOKEN)

memory = {}
modes = {
    "normal": "Обычный режим. Я отвечаю спокойно и понятно.",
    "fun": "Весёлый режим. Я отвечаю с юмором.",
    "smart": "Умный режим. Я отвечаю как мини‑ИИ."
}

current_mode = "normal"

def generate_reply(user_id, text):
    global current_mode
    memory[user_id] = text

    if current_mode == "normal":
        return f"Ты написал: «{text}». Понял тебя."

    if current_mode == "fun":
        return f"Хаха, прикольно ты написал: «{text}». Я бы тоже так сказал!"

    if current_mode == "smart":
        return f"Интересная мысль. Если подумать глубже, то «{text}» звучит довольно логично."

    return "Не понял режим."

@bot.message_handler(commands=['start'])
def start(message):
    markup = types.ReplyKeyboardMarkup(resize_keyboard=True)
    btn1 = types.KeyboardButton("💬 Режимы")
    btn2 = types.KeyboardButton("🧠 Что я говорил?")
    markup.add(btn1, btn2)

    bot.send_message(
        message.chat.id,
        "Привет! Я твой мини‑ChatGPT бот. Пиши что угодно — отвечу.",
        reply_markup=markup
    )

@bot.message_handler(func=lambda m: m.text == "💬 Режимы")
def show_modes(message):
    markup = types.InlineKeyboardMarkup()
    markup.add(types.InlineKeyboardButton("Обычный", callback_data="mode_normal"))
    markup.add(types.InlineKeyboardButton("Весёлый", callback_data="mode_fun"))
    markup.add(types.InlineKeyboardButton("Умный", callback_data="mode_smart"))
    bot.send_message(message.chat.id, "Выбери режим:", reply_markup=markup)

@bot.message_handler(func=lambda m: m.text == "🧠 Что я говорил?")
def show_memory(message):
    user_id = message.from_user.id
    if user_id in memory:
        bot.send_message(message.chat.id, f"Ты писал: «{memory[user_id]}»")
    else:
        bot.send_message(message.chat.id, "Ты ещё ничего не писал.")

@bot.callback_query_handler(func=lambda call: True)
def callback(call):
    global current_mode

    if call.data == "mode_normal":
        current_mode = "normal"
    if call.data == "mode_fun":
        current_mode = "fun"
    if call.data == "mode_smart":
        current_mode = "smart"

    bot.answer_callback_query(call.id, "Режим изменён")
    bot.send_message(call.message.chat.id, f"Режим: {modes[current_mode]}")

@bot.message_handler(func=lambda m: True)
def handle(message):
    user_id = message.from_user.id
    reply = generate_reply(user_id, message.text)
    bot.send_message(message.chat.id, reply)

bot.polling()
