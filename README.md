<img width="1230" alt="image" src="https://github.com/sezaras85/Sentient-ai/blob/main/sentient%20resim.png" />

Bu repoda, **Fireworks AI'nin Dobby-70B modelini** ve **Tweepy API'sini** kullanarak **Twitter için AI destekli bir bot** oluşturmayı öğreneceğiz.  
Bot, **Sentient AI ve yapay zeka** hakkında tweetler paylaşır ve alıntılanan tweetlere **dinamik AI yanıtları** verir.

## 📌 Özellikler

✅ **Fireworks AI’nin Dobby-70B modeli** ile **Sentient AI** hakkında tweet üretir.  
✅ **Günde 2 kez otomatik tweet atar**.  
✅ **5 dakikada bir Twitter’daki alıntı tweetleri kontrol eder** ve AI ile yanıt verir.  
✅ Yanıtlar **dinamik olup, her seferinde farklıdır** ve **200 karakteri geçmez**.  
✅ **Yanıtları terminalde görüntüler**.  
✅ **Ubuntu ve Python üzerinde çalışır** ve **arka planda çalışması için `nohup` kullanılır**.

2️⃣ Twitter API Anahtarlarını ve Fireworks AI API Anahtarını Tanımlayın
config.py dosyanızı oluşturun ve aşağıdaki bilgileri girin:

```bash

BEARER_TOKEN = "TWITTER_BEARER_TOKEN"
API_KEY = "TWITTER_API_KEY"
API_SECRET = "TWITTER_API_SECRET"
ACCESS_TOKEN = "TWITTER_ACCESS_TOKEN"
ACCESS_SECRET = "TWITTER_ACCESS_SECRET"
FIREWORKS_API_KEY = "FIREWORKS_AI_API_KEY"
```

3️⃣ Ana Python Dosyasını (bot.py) Çalıştırın

python bot.py

📝 Kod (bot.py)

import tweepy
import requests
import schedule
import time

# API Anahtarları
from config import BEARER_TOKEN, API_KEY, API_SECRET, ACCESS_TOKEN, ACCESS_SECRET, FIREWORKS_API_KEY

FIREWORKS_API_URL = "https://api.fireworks.ai/inference/v1/completions"

# Twitter API Kimlik Doğrulama
client = tweepy.Client(
    bearer_token=BEARER_TOKEN,
    consumer_key=API_KEY,
    consumer_secret=API_SECRET,
    access_token=ACCESS_TOKEN,
    access_token_secret=ACCESS_SECRET
)

def generate_dynamic_reply(mention_text):
    """ Fireworks AI kullanarak alıntı tweet'e yanıt üretir (200 karakter sınırı). """
    headers = {"Authorization": f"Bearer {FIREWORKS_API_KEY}", "Content-Type": "application/json"}
    prompt = f"Generate a short, safe, and engaging reply. Max 200 characters. Tweet: {mention_text}"
    
    payload = {"model": "accounts/fireworks/models/dobby-70b", "prompt": prompt, "max_tokens": 100, "temperature": 0.7}
    
    response = requests.post(FIREWORKS_API_URL, json=payload, headers=headers)
    if response.status_code == 200:
        ai_reply = response.json()["choices"][0]["text"].strip()
        print(f"Generated AI Reply: {ai_reply}")  # Yanıtı terminalde göster
        return ai_reply[:200]
    else:
        print("Fireworks API Error:", response.json())
        return None

def post_tweet(text):
    """ Tweet gönderme fonksiyonu """
    try:
        response = client.create_tweet(text=text)
        print(f"Tweet posted! Tweet ID: {response.data['id']}")
    except tweepy.TweepyException as e:
        print("Tweet failed:", e)

def send_sentient_ai_tweet():
    """ Sentient AI hakkında bir tweet atar. """
    prompt = "Write an engaging tweet about Sentient AI. Max 200 characters."
    tweet_text = generate_dynamic_reply(prompt)
    if tweet_text:
        post_tweet(tweet_text)

def reply_to_mentions():
    """ Alıntılanan tweetleri kontrol edip yanıt verir. """
    mentions = client.get_mentions()
    for mention in mentions.data:
        reply_text = generate_dynamic_reply(mention.text)
        if reply_text:
            response = client.create_tweet(text=f"@{mention.author_id} {reply_text}", in_reply_to_status_id=mention.id)
            print(f"Reply tweet sent: {response.data['id']}")

# İlk tweet hemen atılsın
send_sentient_ai_tweet()

# Günlük Sentient AI tweetleri (10:00 & 18:00)
schedule.every().day.at("10:00").do(send_sentient_ai_tweet)
schedule.every().day.at("18:00").do(send_sentient_ai_tweet)

# 5 dakikada bir alıntı tweetleri kontrol et ve yanıtla
schedule.every(5).minutes.do(reply_to_mentions)

# Sürekli çalıştır
while True:
    schedule.run_pending()
    time.sleep(60)

Botu Arka Planda Çalıştırma (Ubuntu)
Eğer botu arka planda çalıştırmak istiyorsanız:

nohup python bot.py &







