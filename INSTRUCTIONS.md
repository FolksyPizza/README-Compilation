# PizzaSMP Instructions

Last updated: 2026-02-16

## Paths

- Root: `/ssd/PaperMC/PizzaSMP`
- WebUI backend: `webapp/server.py`
- WebUI static: `webapp/static/`

## Runtime Sessions

- Minecraft server screen: `server`
- WebUI screen: `webui`
- Liberty web screen: `libertyweb`

## Start / Stop

### Start PaperMC (manual)

```bash
screen -S server -dm bash -lc 'cd /ssd/PaperMC/PizzaSMP && ./start.sh'
```

### Stop PaperMC (clean)

```bash
screen -S server -p 0 -X stuff "stop$(printf '\\r')"
```

### Start WebUI

```bash
screen -S webui -X quit || true
screen -dmS webui bash -lc 'cd /ssd/PaperMC/PizzaSMP && python3 webapp/server.py'
```

### Check WebUI health

```bash
curl -sS http://127.0.0.1:3030/api/health
```

## WebUI Notes

- Use **Start** in the panel for the safest PaperMC startup path.
- If the browser shows `NetworkError when attempting to fetch resource`, WebUI is usually down.
- If `python webapp/server.py` prints `OSError: [Errno 98] Address already in use`, stop the existing WebUI process first.

## Port 3030 Recovery

```bash
ss -ltnp | rg ':3030' || true
screen -S webui -X quit || true
pkill -f 'python3 webapp/server.py' || true
```

Then start WebUI again.

## Console UX (current)

- Main console auto-scrolls to bottom.
- Host console auto-scrolls to bottom.
- Stop/restart actions clear console windows to blank before fresh logs return.
