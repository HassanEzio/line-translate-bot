# line-translate-bot
Bot de traduction automatique pour LINE utilisant Flask et l'API LINE Messaging.
# LINE Translate Bot

Bot de traduction automatique pour LINE utilisant Flask et l'API LINE Messaging.

## Installation

```bash
pip install -r requirements.txt
python app.py
Flask==2.3.2
line-bot-sdk==2.0.1
googletrans==4.0.0rc1
__pycache__/
*.pyc
.env
from flask import Flask, request, abort
from linebot import LineBotApi, WebhookHandler
from linebot.exceptions import InvalidSignatureError
from linebot.models import MessageEvent, TextMessage, TextSendMessage
from googletrans import Translator

import os

app = Flask(__name__)

# Récupérer les tokens depuis les variables d'environnement
LINE_CHANNEL_ACCESS_TOKEN = os.getenv('LINE_CHANNEL_ACCESS_TOKEN')
LINE_CHANNEL_SECRET = os.getenv('LINE_CHANNEL_SECRET')

line_bot_api = LineBotApi(LINE_CHANNEL_ACCESS_TOKEN)
handler = WebhookHandler(LINE_CHANNEL_SECRET)
translator = Translator()

@app.route("/callback", methods=['POST'])
def callback():
    signature = request.headers['X-Line-Signature']
    body = request.get_data(as_text=True)
    
    try:
        handler.handle(body, signature)
    except InvalidSignatureError:
        abort(400)
    return 'OK'

@handler.add(MessageEvent, message=TextMessage)
def handle_message(event):
    incoming_text = event.message.text
    # Traduire en anglais par exemple
    translated = translator.translate(incoming_text, dest='en')
    reply_text = translated.text
    
    line_bot_api.reply_message(
        event.reply_token,
        TextSendMessage(text=reply_text)
    )

if __name__ == "__main__":
    app.run(port=8000)
