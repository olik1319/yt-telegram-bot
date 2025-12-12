import os
import logging
from flask import Flask
from telegram import Update
from telegram.ext import Application, MessageHandler, filters, ContextTypes
import yt_dlp
import threading

BOT_TOKEN = os.environ["BOT_TOKEN"]
app = Flask(name)

@app.route("/")
def home():
    return "Bot is running"

logging.basicConfig(level=logging.INFO)

async def handle_link(update: Update, context: ContextTypes.DEFAULT_TYPE):
    url = update.message.text.strip()
    chat_id = update.effective_chat.id

    if not ("youtube.com" in url or "youtu.be" in url):
        await update.message.reply_text("❌ Это не YouTube-ссылка.")
        return

    await update.message.reply_text("⏳ Скачиваю видео...")

    ydl_opts = {
        'outtmpl': '/tmp/%(title)s.%(ext)s',
        'format': 'best',
    }

    try:
        with yt_dlp.YoutubeDL(ydl_opts) as ydl:
            info = ydl.extract_info(url, download=True)
            filename = ydl.prepare_filename(info)

        file_size = os.path.getsize(filename)
        if file_size > 50 * 1024 * 1024:
            await update.message.reply_text("📁 Видео больше 50 МБ, отправляю как файл...")
            with open(filename, 'rb') as file:
                await context.bot.send_document(chat_id=chat_id, document=file)
        else:
            with open(filename, 'rb') as video:
                await context.bot.send_video(chat_id=chat_id, video=video, supports_streaming=True)

        os.remove(filename)

    except Exception as e:
        await update.message.reply_text(f"❌ Ошибка: {e}")

def run_bot():
    application = Application.builder().token(BOT_TOKEN).build()
    application.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_link))
    application.run_polling()

if name == "main":
    threading.Thread(target=run_bot, daemon=True).start()
    port = int(os.environ.get("PORT", 10000))
    app.run(host="0.0.0.0", port=port)
