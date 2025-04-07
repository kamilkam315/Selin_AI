import os
import requests
from telegram import Update
from telegram.ext import ApplicationBuilder, MessageHandler, ContextTypes, filters
import threading
from flask import Flask

# Flask sunucusu
app_flask = Flask('')
@app_flask.route('/')
def home():
    return "Bot aktif!"

def run():
    app_flask.run(host='0.0.0.0', port=8080)

def keep_alive():
    t = threading.Thread(target=run)
    t.start()

# API anahtarların (doğrudan kodda tanımlandı)
GEMINI_API_KEY = "AIzaSyBzP4O2ECrrTSWmwCCA89kxk1qZTzyPzhc"
TELEGRAM_TOKEN = "7767482026:AAE1OENYthBNggtI-lTYf0HWRsmTAxyy8CE"

async def handle_message(update: Update, context: ContextTypes.DEFAULT_TYPE):
    user_message = update.message.text
    url = f"https://generativelanguage.googleapis.com/v1beta/models/gemini-1.5-flash:generateContent?key={GEMINI_API_KEY}"
    headers = {"Content-Type": "application/json"}
    data = {
        "contents": [
            {
                "parts": [{"text": f"Lütfen sadece Türkçe cevap ver. Kullanıcının mesajı: {user_message}"}]
            }
        ]
    }

    try:
        response = requests.post(url, headers=headers, json=data)
        result = response.json()
        cevap = result["candidates"][0]["content"]["parts"][0]["text"]
    except Exception as e:
        cevap = f"Hata oluştu: {e}"

    await update.message.reply_text(f"Selin: {cevap}")

if __name__ == "__main__":
    keep_alive()
    print("Selin Telegram botu Railway'de çalışıyor...")
    app = ApplicationBuilder().token(TELEGRAM_TOKEN).build()
    app.add_handler(MessageHandler(filters.TEXT & ~filters.COMMAND, handle_message))
    app.run_polling()
