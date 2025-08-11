"""
signal_bot.py
Signal-only Quotex watcher -> Telegram notifier
NOTE: Read-only by default. Test on demo account first.
"""

import os
import time
import pandas as pd
import requests
from dotenv import load_dotenv

# Try import of common community libraries (name may differ)
try:
    # preferred community libs
    from quotexpy import Quotex         # repo: SantiiRepair/quotexpy
except Exception:
    try:
        from pyquotex import PyQuotex   # alternative: cleitonleonel/pyquotex
    except Exception:
        raise SystemExit("Install quotexpy or pyquotex (see README).")

load_dotenv()

# ---------- CONFIG ----------
USERNAME = os.getenv("QUOTEX_EMAIL")        # or session token depending on library
PASSWORD = os.getenv("QUOTEX_PASSWORD")
USE_DEMO = True                              # ensure demo mode for safety
TELEGRAM_BOT_TOKEN = os.getenv("TELEGRAM_BOT_TOKEN")
TELEGRAM_CHAT_ID = os.getenv("TELEGRAM_CHAT_ID")

SYMBOL = "EURUSD"     # change to asset supported on Quotex
POLL_INTERVAL = 3     # seconds (be reasonable to avoid rate limits)
EMA_SHORT = 12
EMA_LONG = 26

# ---------- Helpers ----------
def send_telegram(text):
    if not TELEGRAM_BOT_TOKEN or not TELEGRAM_CHAT_ID:
        print("Telegram not configured. Message:", text)
        return
    url = f"https://api.telegram.org/bot{TELEGRAM_BOT_TOKEN}/sendMessage"
    requests.post(url, json={"chat_id": TELEGRAM_CHAT_ID, "text": text}, timeout=10)

def compute_emas(prices):
    s = pd.Series(prices)
    ema_s = s.ewm(span=EMA_SHORT, adjust=False).mean().iloc[-1]
    ema_l = s.ewm(span=EMA_LONG, adjust=False).mean().iloc[-1]
    prev_ema_s = s.ewm(span=EMA_SHORT, adjust=False).mean().iloc[-2]
    prev_ema_l = s.ewm(span=EMA_LONG, adjust=False).mean().iloc[-2]
    # crossover detection
    if ema_s > ema_l and prev_ema_s <= prev_ema_l:
        return 1  # CALL / BUY signal
    if ema_s < ema_l and prev_ema_s >= prev_ema_l:
        return -1 # PUT / SELL signal
    return 0

# ---------- Connect to Quotex (read-only) ----------
def connect_quotex():
    # Pseudocode — actual login depends on the chosen library.
    # Example for quotexpy: client = Quotex(email, password, demo=USE_DEMO)
    # Example for pyquotex: client = PyQuotex(session_token=...)
    # We'll attempt a common pattern and instruct you in README to adjust.
    try:
        client = Quotex()
        client.login(USERNAME, PASSWORD)   # library-specific
        if USE_DEMO:
            client.use_demo_mode()
        return client
    except Exception:
        # try alternate client name
        client = PyQuotex()
        client.login(USERNAME, PASSWORD)
        return client

def fetch_recent_prices(client, symbol, n=100):
    # library-specific: many provide a way to fetch recent ticks or candles
    # adapt below to match library API. Example function names: get_ticks, get_history
    data = []
    try:
        ticks = client.get_ticks(symbol, count=n)   # example only
        for t in ticks:
            data.append(float(t['price']))
    except Exception:
        # fallback: try candles
        candles = client.get_candles(symbol, timeframe='M1', count=n)
        for c in candles:
            data.append(float(c['close']))
    return data

# ---------- Main loop ----------
def main():
    send_telegram("Signal-only bot starting (demo=%s)" % USE_DEMO)
    client = connect_quotex()
    last_signal = 0

    while True:
        try:
            prices = fetch_recent_prices(client, SYMBOL, n=200)
            if len(prices) < max(EMA_LONG+2, 60):
                time.sleep(POLL_INTERVAL)
                continue

            sig = compute_emas(prices)
            price_now = prices[-1]

            if sig == 1 and last_signal != 1:
                txt = f"CALL signal for {SYMBOL} @ {price_now}\nStrategy: EMA{EMA_SHORT}/{EMA_LONG}"
                send_telegram(txt); print(txt)
                last_signal = 1
            elif sig == -1 and last_signal != -1:
                txt = f"PUT signal for {SYMBOL} @ {price_now}\nStrategy: EMA{EMA_SHORT}/{EMA_LONG}"
                send_telegram(txt); print(txt)
                last_signal = -1
            # else no new signal

        except Exception as e:
            print("Loop error:", e)
            send_telegram(f"Signal bot error: {e}")

        time.sleep(POLL_INTERVAL)

if __name__ == "__main__":
    main() align="center">
<img src="https://static.scarf.sh/a.png?x-pxid=cf317fe7-2188-4721-bc01-124bb5d5dbb2" />

## <img src="https://github.com/SantiiRepair/quotexpy/blob/main/.github/images/quotex-logo.png?raw=true" height="56"/>


**📈 QuotexPy is a library to easily interact with qxbroker.**

______________________________________________________________________

[![License](https://img.shields.io/badge/License-LGPL--2.1-magenta.svg)](https://www.gnu.org/licenses/old-licenses/lgpl-2.1.txt)
[![PyPI version](https://badge.fury.io/py/quotexpy.svg)](https://badge.fury.io/py/quotexpy)
![GithubActions](https://github.com/SantiiRepair/quotexpy/actions/workflows/pylint.yml/badge.svg)

</div>

______________________________________________________________________

## Installing

📈 QuotexPy is tested on Ubuntu 18.04 and Windows 10 with **Python >= 3.10, <= 3.12.**
```bash
pip install quotexpy
```

If you plan to code and make changes, clone and install it locally.

```bash
git clone https://github.com/SantiiRepair/quotexpy.git
pip install -e .
```

## Import
```python
from quotexpy import Quotex
```

## Examples
For examples check out [some](https://github.com/SantiiRepair/quotexpy/blob/main/example/main.py) found in the `example` directory.

## Donations
If you feel like showing your love and/or appreciation for this project, then how about shouting us a coffee ;)

[![ko-fi](.github/images/ko-fi.svg)](https://ko-fi.com/SantiiRepair)

## Acknowledgements
- Thanks to [@cleitonleonel](https://github.com/cleitonleonel) for the initial base implementation of the project 🔥
- Thanks to [@ricardospinoza](https://github.com/ricardospinoza) for solving the `trade` error in the code 🚀

## Notice 
This project is a clone of the [original](https://github.com/cleitonleonel/pyquotex) project, because the original project was discontinued, I updated it with the help of [collaborators](https://github.com/SantiiRepair/quotexpy/graphs/contributors) in the community so that it is accessible to everyone.
