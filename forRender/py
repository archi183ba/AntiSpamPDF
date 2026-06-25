import os
import threading
import re
from flask import Flask
from telegram import Update, InlineKeyboardButton, InlineKeyboardMarkup
from telegram.ext import Application, CommandHandler, MessageHandler, filters, CallbackQueryHandler

app = Flask(__name__)


TOKEN = os.environ.get("TELEGRAM_TOKEN")


PREFIXES = []
REGEX_PATTERNS = []
EXACT_WORDS = []
WHITELIST = []

def load_banned_words():
    global PREFIXES, REGEX_PATTERNS, EXACT_WORDS, WHITELIST
    
    
    PREFIXES = []
    REGEX_PATTERNS = []
    EXACT_WORDS = []
    WHITELIST = []
    
    try:
        with open('banned_words.txt', 'r', encoding='utf-8') as f:
            for line in f:
                word = line.strip().lower()
                if not word or word.startswith('#'):  # Пропускаем пустые строки и комментарии
                    continue
                
                # Определяем тип слова
                if any(char in word for char in '[].*+?^$|(){}'):
                    # Если есть спецсимволы — это регулярное выражение
                    REGEX_PATTERNS.append(word)
                elif len(word) <= 3:
                    # Короткие слова — как префиксы
                    PREFIXES.append(word)
                else:
                    # Длинные слова — как точные
                    EXACT_WORDS.append(word)
    except FileNotFoundError:
        print("⚠️ Файл banned_words.txt не найден, использую стандартные настройки")
        # Стандартные настройки на случай отсутствия файла
        PREFIXES = ["пед", "пид", "ped", "pid", "пдф", "pdf"]
        EXACT_WORDS = ["педофил", "педик", "pedophile"]
    
    # Загружаем белый список (если есть)
    try:
        with open('whitelist.txt', 'r', encoding='utf-8') as f:
            for line in f:
                word = line.strip().lower()
                if word and not word.startswith('#'):
                    WHITELIST.append(word)
    except FileNotFoundError:
        print("ℹ️ Файл whitelist.txt не найден, белый список пуст")
    
    print(f"✅ Загружено префиксов: {len(PREFIXES)}")
    print(f"✅ Загружено регулярок: {len(REGEX_PATTERNS)}")
    print(f"✅ Загружено точных слов: {len(EXACT_WORDS)}")
    print(f"✅ Загружено слов в белый список: {len(WHITELIST)}")

def is_banned(text: str) -> bool:
    """Проверяет, содержит ли текст запрещённые слова"""
    if not text:
        return False
    
    text_lower = text.lower()
    
    # 1. Проверка белого списка (пропускаем)
    for word in WHITELIST:
        if word in text_lower:
            return False
    
    # 2. Проверка префиксов
    for prefix in PREFIXES:
        for word in text_lower.split():
            if word.startswith(prefix):
                return True
    
    # 3. Проверка регулярных выражений
    for pattern in REGEX_PATTERNS:
        if re.search(pattern, text_lower, re.IGNORECASE):
            return True
    
    # 4. Проверка точных слов
    for word in EXACT_WORDS:
        if word in text_lower:
            return True
    
    return False

# Команды бота 
async def start(update: Update, context):
    keyboard = [
        [InlineKeyboardButton("🔄 Перезагрузить список", callback_data='reload')],
        [InlineKeyboardButton("📊 Статистика", callback_data='stats')]
    ]
    reply_markup = InlineKeyboardMarkup(keyboard)
    
    await update.message.reply_text(
        "👋 Привет! Я бот-фильтр.\n\n"
        "📌 Я блокирую сообщения с запрещёнными словами.\n"
        "📁 Список слов хранится в файле banned_words.txt\n"
        "⚙️ Добавьте меня в группу как администратора!\n\n"
        "Нажмите кнопку ниже, чтобы перезагрузить список после редактирования файла.",
        reply_markup=reply_markup
    )

async def reload_words(update: Update, context):
    """Перезагружает списки из файлов"""
    load_banned_words()
    await update.callback_query.answer("✅ Список обновлён!")
    await update.callback_query.edit_message_text(
        "✅ Списки слов успешно перезагружены!\n\n"
        f"📊 Префиксов: {len(PREFIXES)}\n"
        f"📊 Регулярных выражений: {len(REGEX_PATTERNS)}\n"
        f"📊 Точных слов: {len(EXACT_WORDS)}\n"
        f"📊 Белый список: {len(WHITELIST)}"
    )

async def stats(update: Update, context):
    """Показывает статистику"""
    await update.callback_query.answer()
    await update.callback_query.edit_message_text(
        "📊 Текущая статистика:\n\n"
        f"🔸 Префиксов: {len(PREFIXES)}\n"
        f"🔸 Регулярных выражений: {len(REGEX_PATTERNS)}\n"
        f"🔸 Точных слов: {len(EXACT_WORDS)}\n"
        f"🔸 Белый список: {len(WHITELIST)}\n"
    )

async def filter_message(update: Update, context):
    text = update.message.text
    if text and is_banned(text):
...         try:
...             await update.message.delete()
...             await update.message.reply_text("⛔ Сообщение удалено за нарушение правил чата.")
...         except Exception as e:
...             print(f"Ошибка при удалении: {e}")
... 
... def run_bot():
...     """Запускает бота в отдельном потоке"""
...     application = Application.builder().token(TOKEN).build()
...     
...     # Команды
...     application.add_handler(CommandHandler("start", start))
...     application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, filter_message))
...     
...     # Кнопки
...     application.add_handler(CallbackQueryHandler(reload_words, pattern='reload'))
...     application.add_handler(CallbackQueryHandler(stats, pattern='stats'))
...     
...     application.run_polling()
... 
... # Flask для Render
... @app.route('/')
... def hello():
...     return "Bot is running"
... 
... @app.route('/health')
... def health():
...     return "OK"
... 
... if __name__ == '__main__':
...     # Загружаем списки при старте
...     load_banned_words()
...     
...     # Запускаем бота
...     bot_thread = threading.Thread(target=run_bot)
...     bot_thread.start()
...     
...     # Запускаем Flask
...     port = int(os.environ.get("PORT", 5000))
