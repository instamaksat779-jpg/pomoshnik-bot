import os
import logging
import nest_asyncio
nest_asyncio.apply()
from dotenv import load_dotenv
from flask import Flask, request
from telegram import Update, ReplyKeyboardMarkup, KeyboardButton, InlineKeyboardMarkup, InlineKeyboardButton
from telegram.ext import ApplicationBuilder, CommandHandler, MessageHandler, CallbackQueryHandler, filters, ContextTypes
from ai import full_discussion

load_dotenv()
TELEGRAM_TOKEN = os.getenv("TELEGRAM_TOKEN")
PORT = int(os.environ.get("PORT", 8080))
RENDER_EXTERNAL_URL = os.environ.get("RENDER_EXTERNAL_URL")
if not RENDER_EXTERNAL_URL:
    RENDER_EXTERNAL_URL = "https://your-render-url.onrender.com"

logging.basicConfig(level=logging.INFO)

MENU = ReplyKeyboardMarkup([
    [KeyboardButton("💬 Задать вопрос")],
    [KeyboardButton("ℹ️ О боте"), KeyboardButton("🆘 Помощь")]
], resize_keyboard=True)

async def start(update: Update, context: ContextTypes.DEFAULT_TYPE):
    keyboard = InlineKeyboardMarkup([
        [InlineKeyboardButton("🚀 Поехали!", callback_data="start_bot")]
    ])
    await update.message.reply_text(
        "👋 Привет! Добро пожаловать в Pomoshnik Bot!\n\n🤖 Я твой личный AI-помощник!\n\n👨‍💻 Разработчик: Макс\n\nНажми кнопку чтобы начать 👇",
        reply_markup=keyboard
    )

async def button(update: Update, context: ContextTypes.DEFAULT_TYPE):
    query = update.callback_query
    await query.answer()
    await query.message.reply_text("🎉 Отлично! Я готов помочь!\n\nПросто напиши мне любой вопрос!", reply_markup=MENU)

async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    text = update.message.text
    if text == "ℹ️ О боте":
        await update.message.reply_text("🤖 Pomoshnik Bot\n\nЯ AI-помощник!\n👨‍💻 Создатель: Макс\n⚡️ Работает на базе Llama 3.3", reply_markup=MENU)
    elif text == "🆘 Помощь":
        await update.message.reply_text("💬 Просто напиши мне любой вопрос!\n\nНапример:\n• Что такое ChatGPT?\n• Как выучить Python?\n• Расскажи анекдот", reply_markup=MENU)
    else:
        await update.message.reply_text("⏳ Думаю...")
        response = full_discussion(text)
        await update.message.reply_text(response, reply_markup=MENU)

flask_app = Flask(__name__)
telegram_app = None

@flask_app.route("/webhook", methods=["POST"])
async def webhook():
    """Принимает обновления от Telegram."""
    if telegram_app is None:
        return "telegram_app not initialized", 500
    update = Update.de_json(request.get_json(force=True), telegram_app.bot)
    await telegram_app.process_update(update)
    return "ok"

async def set_webhook(app):
    """Установить вебхук на указанный URL."""
    webhook_url = f"{RENDER_EXTERNAL_URL}/webhook"
    await app.bot.set_webhook(webhook_url)
    logging.info(f"Webhook установлен: {webhook_url}")

async def main():
    global telegram_app
    telegram_app = ApplicationBuilder().token(TELEGRAM_TOKEN).build()
    telegram_app.add_handler(CommandHandler("start", start))
    telegram_app.add_handler(CallbackQueryHandler(button))
    telegram_app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))

    await telegram_app.initialize()
    await set_webhook(telegram_app)
    logging.info("Бот запущен с вебхуком!")

if __name__ == "__main__":
    import asyncio
    asyncio.get_event_loop().run_until_complete(main())
    flask_app.run(host="0.0.0.0", port=PORT)
