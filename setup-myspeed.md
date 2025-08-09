# Table of Contents
- [Setup MySpeed Service](#setup-myspeed-service)

# Setup MySpeed Service
1. Go to `Portainer` and create a new stack with the following configuration:

```yaml
version: '3.8'

services:
  myspeed:
    image: germannewsmaker/myspeed
    container_name: MySpeed
    ports:
      - "5216:5216"
    restart: unless-stopped
```

2. Once the stack is created, you can access the MySpeed web interface by navigating to `http://your-server-ip:5216` in your web browser. (If you have set your public domain via Cloudflare, you can access it via say `https://myspeed.yourdomain.com`)