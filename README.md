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

Öncelikle, sisteminin aşağıdaki gereksinimlere sahip olduğundan emin ol:

✅ Ubuntu 20.04+
✅ Python 3.8+
✅ pip & venv (Python sanal ortamı)
✅ API erişim anahtarı (Fireworks AI platformundan alman gerekebilir)

1. Twitter API Anahtarlarını Al
Öncelikle Twitter Developer Portal üzerinden bir Twitter API hesabı oluştur ve aşağıdaki bilgileri al:

API Key
API Key Secret
Access Token
Access Token Secret
Bearer Token
Bu bilgileri aldıktan sonra projemizde kullanacağız.

Eğer pip ve venv yüklü değilse, terminalde şu komutları çalıştırarak yükleyebilirsin:
```bash
sudo apt update && sudo apt upgrade -y
sudo apt install python3 python3-pip python3-venv -y
pip install tweepy requests python-dotenv schedule

```

3. .env Dosyasını Oluştur ve API Anahtarlarını Sakla
```bash
nano .env
```

Şu satırları ekle (kendi API bilgilerinle değiştir):
```bash
TBEARER_TOKEN = "BEARER_TOKEN YAZ"
API_KEY = "API_KEY YAZ"
API_SECRET = "API_SECRET YAZ"
ACCESS_TOKEN = "ACCESS_TOKEN YAZ"
ACCESS_SECRET = "ACCESS_SECRET YAZ"
```
CTRL + X, sonra Y ve Enter ile kaydet.

4. Twitter Bot Kodunu Yaz
Şimdi twitter_bot.py dosyasını oluştur ve şu kodları ekle:
```bash
import tweepy
import requests
import schedule
import time

# Twitter API Keys
BEARER_TOKEN = "BEARER_TOKEN"
API_KEY = "API_KEY"
API_SECRET = "API_SECRET"
ACCESS_TOKEN = "ACCESS_TOKEN"
ACCESS_SECRET = "ACCESS_SECRET"

# Fireworks AI API Key
FIREWORKS_API_KEY = "API KEY"
FIREWORKS_API_URL = "https://api.fireworks.ai/inference/v1/completions"

# Twitter API Authentication
client = tweepy.Client(
    bearer_token=BEARER_TOKEN,
    consumer_key=API_KEY,
    consumer_secret=API_SECRET,
    access_token=ACCESS_TOKEN,
    access_token_secret=ACCESS_SECRET
)

def generate_dynamic_reply(mention_text):
    """ Generates a dynamic reply using Fireworks AI based on the mention text. """
    headers = {
        "Authorization": f"Bearer {FIREWORKS_API_KEY}",
        "Content-Type": "application/json"
    }
    
    # Create a prompt based on the mention text
    prompt = f"Generate a short, positive, and safe response to the following tweet. Maximum 200 characters. Here is the tweet: {mention_text}"
    
    payload = {
        "model": "accounts/sentientfoundation/models/dobby-unhinged-llama-3-3-70b-new",
        "prompt": prompt,
        "max_tokens": 100,
        "temperature": 0.7,
        "top_p": 0.9
    }
    
    response = requests.post(FIREWORKS_API_URL, json=payload, headers=headers)
    
    if response.status_code == 200:
        ai_reply = response.json()["choices"][0]["text"].strip()
        print(f"Generated AI Reply: {ai_reply}")  # Print the AI-generated reply to the console
        return ai_reply[:200]  # Limit to 200 characters
    else:
        print("Fireworks API Error:", response.json())
        return None

def post_tweet(text):
    """ Posts a tweet using Twitter API. """
    try:
        response = client.create_tweet(text=text)
        print(f"Tweet posted! Tweet ID: {response.data['id']}")
    except tweepy.TweepyException as e:
        print("Tweet failed:", e)

def send_sentient_ai_tweet():
    """ Posts an AI-generated tweet about Sentient AI in English. """
    prompt = "Write an interesting, positive, and safe tweet about Sentient AI. Maximum 200 characters. The tweet should be in English."
    tweet_text = generate_dynamic_reply(prompt)  # Dynamically generated tweet content
    if tweet_text:
        post_tweet(tweet_text)

def reply_to_mentions():
    """ Checks mentions and replies to quotes and mentions using Fireworks AI. """
    mentions = client.get_mentions()
    
    # Loop through mentions and reply to the quotes
    for mention in mentions.data:
        if "quoted_status" in mention.text:  # Check if it's a quote tweet
            reply_text = generate_dynamic_reply(mention.text)  # Generate dynamic reply based on the mention text
            if reply_text:
                response = client.create_tweet(
                    text=f"@{mention.author_id} {reply_text}",
                    in_reply_to_status_id=mention.id  # This replies to the quoted tweet
                )
                print(f"Reply tweet sent: {response.data['id']}")

# Post the first tweet immediately
send_sentient_ai_tweet()

# Schedule 2 automatic Sentient AI tweets daily (10:00 AM and 6:00 PM)
schedule.every().day.at("10:00").do(send_sentient_ai_tweet)
schedule.every().day.at("18:00").do(send_sentient_ai_tweet)

# Check mentions and reply to them every 5 minutes
schedule.every(5).minutes.do(reply_to_mentions)

# Keep the bot running
while True:
    schedule.run_pending()
    time.sleep(60)  # Wait 1 minute
```
5. Twitter Botunu Çalıştır
   Şimdi terminalde botu başlat:
 ```bash  
 python3 twitter_bot.py
```
Bot artık alıntılanan tweetlere cevap verecek!
✅ Her gün 10:00 ve 18:00’de Sentient AI hakkında tweet atacak!

Botu arka planda çalıştırmak için:

```bash
nohup python3 twitter_bot.py &
```

SONUÇ: Sentient AI Twitter Botun Hazır!
🔹 Alıntı yapılan tweetlere otomatik cevap veriyor.
🔹 Günde 2 defa Sentient AI hakkında tweet atıyor.
🔹 Fireworks AI kullanarak doğal dilde yanıt oluşturuyor.








  







   
   
















