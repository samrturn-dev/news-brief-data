# Daily news brief (Rasp5)

A morning summary of global news, markets, sport, science/climate and good news. It's built on the Pi at 05:45 UTC and ready by **06:00 GMT**. It arrives as an **email** and as a **Scriptable** list and widget.

## How it works
1. `brief.py` pulls ~26 RSS feeds. These include paywalled ones (FT, Economist, Bloomberg, WSJ) for their headlines.
2. Claude **Haiku 5.5** groups stories, keeps politics to 3 or fewer, and writes 2–3 sentences per story. It refers to stories by id only, so it never writes URLs.
3. The code picks each story's link. A free source is used whenever one covers the same event; otherwise the paywalled link is shown with 🔒, along with an "also covered by" note.
4. Market figures come from yfinance: FTSE, SMI, S&P, GBP/CHF, GBP/USD, gold and BTC.
5. Outputs: `archive/YYYY-MM-DD.json` and `brief.html`. The brief is emailed through Gmail, and `brief.json` is pushed to GitHub for Scriptable to read.
6. The script skips anything covered yesterday unless there's a significant update.

Sport priorities: Aston Villa; England and Australia in football, rugby union and cricket; Premier League; Champions League; golf.

Cost: about 30k tokens in and 3k out per day with Haiku 5.5, which comes to about $0.005/day (around $2/year).

## Setup on the Pi (user `samt`)
```bash
mkdir -p ~/news-brief && cd ~/news-brief    # copy the files here
python3 -m venv .venv && .venv/bin/pip install -r requirements.txt
cat > .env <<'EOF'
ANTHROPIC_API_KEY=sk-ant-...
GMAIL_USER=you@gmail.com
GMAIL_APP_PASSWORD=xxxx xxxx xxxx xxxx
EOF
chmod 600 .env
.venv/bin/python brief.py --check-feeds   # fix or remove any ✗ feeds in config.yaml
.venv/bin/python brief.py --dry-run       # builds brief.html, no email/push
```
Then edit `email.to` in `config.yaml` to your address.

**GitHub (for Scriptable):** create a repo called `news-brief-data` and clone it to `/home/samt/news-brief-data`. Give the Pi push access with a deploy key. Set `BRIEF_URL` in `scriptable/NewsBrief.js`. If the repo is private, also set a read-only token there.

**Schedule:**
```bash
sudo cp systemd/news-brief.* /etc/systemd/system/
sudo systemctl daemon-reload && sudo systemctl enable --now news-brief.timer
systemctl list-timers news-brief.timer      # confirm next run
journalctl -u news-brief -n 50              # logs
```
The schedule is fixed at 06:00 GMT, so in summer it arrives at **07:00 BST**. To keep it at 6am local time all year, change the timer to `OnCalendar=*-*-* 05:45:00 Europe/London`.

## Ideas for later
- Fixtures and results from football-data.org (free key) for Villa, the Premier League and the Champions League
- 👍/👎 feedback from Scriptable to tune the story selection
- Weekend edition with longer reads
- Ollama side-by-side test on the Mac
