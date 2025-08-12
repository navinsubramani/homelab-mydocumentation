# Table of Contents
- [Setup Watchtower](#setup-watchtower)

# Setup Watchtower
Watchtower is a tool that automatically updates running Docker containers whenever a new image is available. This is particularly useful for keeping your services up-to-date without manual intervention.

Use the below compose to run Watchtower in your Docker environment:

```yaml
version: "3"
services:
  watchtower:
    container_name: WatchTower
    image: containrrr/watchtower
    restart: always
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /etc/timezone:/etc/timezone:ro
    ports:
      - "xxxx:8080"
    environment:
      TZ: America/New_York
      WATCHTOWER_CLEANUP: true
      WATCHTOWER_INCLUDE_RESTARTING: true
      WATCHTOWER_INCLUDE_STOPPED: true
      WATCHTOWER_SCHEDULE: "0 0 * * *"
      WATCHTOWER_NOTIFICATIONS: shoutrrr
      WATCHTOWER_NOTIFICATION_URL: ${DISCORD}
      # Discord shoutrr format: discord://WEBHOOK_TOKEN@WEBHOOK_ID
      WATCHTOWER_HTTP_API_UPDATE: "true"
      WATCHTOWER_HTTP_API_METRICS: "true"
      WATCHTOWER_HTTP_API_TOKEN: ${TOKEN}
      WATCHTOWER_HTTP_API_PERIODIC_POLLS: "true"
```

Environment variables:
- `DISCORD`: The Discord webhook URL for notifications.
- `TOKEN`: A token for the Watchtower HTTP API, which allows you to interact with Watchtower via HTTP requests.

This configuration should be done in each docker environment where you want to run Watchtower. The `WATCHTOWER_SCHEDULE` environment variable is set to run daily at midnight, but you can adjust it according to your needs.