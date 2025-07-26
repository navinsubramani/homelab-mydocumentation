
# Table of Contents
- [Setup Pairdrop](#setup-pairdrop)


# Setup Pairdrop
1. Go to `Portainer` and create a new stack with the following configuration:

```yaml
version: "3"
services:
  pairdrop:
    image: "lscr.io/linuxserver/pairdrop:latest"
    container_name: pairdrop
    restart: unless-stopped
    environment:
      - PUID=${PUID} # UID to run the application as
      - PGID=${PGID} # GID to run the application as
      - WS_FALLBACK=true # Set to true to enable websocket fallback if the peer to peer WebRTC connection is not available to the client.
      - PUID=true # Set to true to limit clients to 1000 requests per 5 min.
      - RTC_CONFIG=false # Set to the path of a file that specifies the STUN/TURN servers.
      - DEBUG_MODE=false # Set to true to debug container and peer connections.
      - TZ=America/New_York # Time Zone
    ports:
      - "3001:3000" # Web UI. Change the port number before the last colon e.g. `127.0.0.1:9000:3000`
```

2. Instead of 1, you can also the documentation of Pairdrop to set up the service using [this](https://github.com/beingofexistence13/pairdrop) link.

3. Once the stack is created, you can access the Pairdrop web interface by navigating to `http://your-server-ip:3000` in your web browser. (If you have set your public domain via Cloudflare, you can access it via say `https://pairdrop.yourdomain.com`)

_NOTE_: If you are in local network, the peer will be directly recignized and you can transfer files directly without going through the server.

_NOTE_: If you are outside the local network, the peer will not be recognized automatically as they are in different networks. You can still transfer files by first pairing the deveices using the pairdrop web interface. Once paired, you can transfer files between the devices.
