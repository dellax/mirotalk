# MiroTalk SFU - Docker Setup

Lightweight self-hosted video conferencing with RTMP streaming support.

## Requirements

- Docker + Docker Compose
- Open ports: `3010/tcp` (web UI), `1935/tcp` (RTMP), `40000-40100/tcp+udp` (WebRTC media)

## Quick Start

1. **Configure `.env`:**
   - Set `SFU_ANNOUNCED_IP` to your server's public IP
   - Set `SERVER_HOST_URL` to your domain (e.g., `https://meet.example.com`)
   - Set admin credentials in `HOST_USERS` (see Admin section below)

2. **Create directories and start:**
   ```bash
   mkdir -p rec rtmp
   docker compose up -d
   ```

3. **Access:** Open `http://your-server-ip:3010` in a browser.

## Admin / Host Controls

MiroTalk SFU has built-in admin features — no extra services needed.

### Enable host protection

Set in `.env`:
```
HOST_PROTECTED=true
HOST_USER_AUTH=true
HOST_USERS="admin:yourpassword:Admin:*"
```

Format: `username:password:displayName:allowedRooms` (`*` = all rooms).

Multiple users: separate with `|`:
```
HOST_USERS="admin:strongpass:Admin:*|presenter:pass123:Speaker:*"
```

### What the host/presenter can do

- **Room lobby** — approve or reject who joins (`ROOM_LOBBY=true` or toggle in-room)
- **Lock room** — prevent new participants from joining
- **Mute/kick participants** — individual or all at once
- **Moderator tab** — full control panel in Settings during a meeting
- **Start/stop recording** — server-side, saved to `./rec/`
- **Start/stop RTMP stream** — to YouTube, Twitch, Facebook, or any custom RTMP URL
- **Polls** — create and manage polls in real-time
- **Whiteboard** — lock/unlock for participants
- **Broadcasting** — one-to-many mode

### What guests can do

- Join rooms created by the host (via shared link)
- Use audio, video, screen share, chat
- Cannot create rooms, access moderator controls, or start recordings/streams

## Streaming to YouTube Live

1. In a MiroTalk room, open **Settings > RTMP tab**
2. Select "Custom URL" and paste your YouTube RTMP URL + stream key
   - YouTube Studio > Go Live > Stream > copy stream URL + key
3. Click start — audience watches on YouTube (unlimited viewers)
4. Stop streaming from the same tab when done

The `mirotalk-nms` container handles RTMP processing via Node Media Server.

## TLS / Reverse Proxy

MiroTalk runs on port `3010` (HTTP). For HTTPS, use a reverse proxy:

### Caddy (simplest)
```
meet.example.com {
    reverse_proxy localhost:3010
}
```

### Nginx
```nginx
server {
    server_name meet.example.com;
    location / {
        proxy_pass http://localhost:3010;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection "upgrade";
        proxy_set_header Host $host;
        proxy_http_version 1.1;
    }
}
```

## Capacity (5GB RAM server)

| Mode | Max users |
|------|-----------|
| Audio-only participants | ~100-200 |
| Webcam on | ~30-50 per CPU core |
| Speakers with cam + audience audio-only | ~100+ |
| Streaming to YouTube (audience on YT) | Unlimited audience |

Room limit is set to 100 (`ROOM_MAX_PARTICIPANTS` in `.env`).

## Customization

### Branding (`.env`)
```
APP_NAME=Your Conference Name
SITE_TITLE=Your Conference
```

MiroTalk branding is already hidden in this setup (`SHOW_SPONSORS=false`, `SHOW_FOOTER=false`, etc.).

### Features

Toggle any button or panel via `SHOW_*` variables in `.env`:
```
SHOW_POLL_BUTTON=true       # Polls
SHOW_WHITEBOARD=true        # Whiteboard
SHOW_RTMP_TAB=true          # RTMP streaming tab
SHOW_RECORDING_TAB=true     # Recording tab
SHOW_LOBBY=true             # Lobby controls
SHOW_VIRTUAL_BACKGROUND=true
```

### Advanced

Edit `app/src/config.js` for full control over defaults and logic.

## Useful Commands

```bash
# Start
docker compose up -d

# Stop
docker compose down

# View logs
docker compose logs -f mirotalksfu

# View RTMP server logs
docker compose logs -f mirotalk-nms

# Restart
docker compose restart

# Update to latest
docker compose pull && docker compose up -d
```

## Files

```
mirotalk/
├── docker-compose.yml     # Services: mirotalksfu + mirotalk-nms (RTMP)
├── .env                   # All configuration
├── app/src/config.js      # Advanced config (defaults from template)
├── rec/                   # Server-side recordings
├── rtmp/                  # RTMP file sources
└── README.md
```

## Version

MiroTalk SFU v2.1.73 (latest Docker image)
