# Table of Contents
- [Requirements](#requirements)
- [Setup Cloudflare and Configure the Domain](#setup-cloudflare-and-configure-the-domain)
- [Setup the SSL/TLS](#setup-the-ssltls)
- [Setup Cloudflare Tunnel](#setup-cloudflare-tunnel)
- [Setup Cloudflare Service on your Server](#setup-cloudflare-service-on-your-server)
- [Test the Clouflare Tunnel Connections](#test-the-clouflare-tunnel-connections)
- [Configure Securtity Settings using Application and Policies](#configure-securtity-settings-using-application-and-policies)
- [Test the Cloudflare Access Application](#test-the-cloudflare-access-application)


_NOTE_: Cloudflare Tunnel is a service that allows you to securely expose your local services to the internet without exposing your server's IP address. It creates a secure tunnel between your server and Cloudflare's edge network, allowing you to access your services securely over HTTPS. This is a good option instead of exposing your local network IP and ports to public, and also buying a static IP address from your ISP. It also provides additional security features like DDoS protection, SSL/TLS encryption, and access control.


# Requirements
1. A domain name. You can by a domain name from any domain registrar like [Namecheap](https://www.namecheap.com/), [GoDaddy](https://www.godaddy.com/), etc.
2. A Cloudflare account. You can sign up for free at [Cloudflare](https://www.cloudflare.com/).

# Setup CLoudflare and Configure the Domain
1. Sign in to your Cloudflare account.
2. Add your domain to Cloudflare by clicking on the "Add a domain" button.
3. Follow the instructions to change your domain's nameservers (on Domain Registrar) to the ones provided by Cloudflare.

_NOTE_: Make sure to wait for the DNS changes to propagate, which can take a few minutes to a few hours depending on your domain registrar.

4. Once your domain is added, go to the "DNS" tab in the Cloudflare dashboard to confirm that your domain's DNS records are set up correctly.

# Setup the SSL/TLS
1. In the Cloudflare dashboard for your domain, go to the "SSL/TLS" tab.
2. Under `Overview`, set the SSL/TLS encryption mode to `Full (strict)` for better security.
3. Under `Edge Certificates`, enable the following options:
   - Always Use HTTPS
   - Automatic HTTPS Rewrites
   - Enable Universal SSL


# Setup Cloudflare Tunnel
1. In the Cloudflare dashboard, go to the "Zero Trust" tab. (You can find this under the `Access` section)
2. Click on `Networks` -> `Tunnels` in the left sidebar.
3. Click on the `Create a tunnel` button.
4. Enter a name for your tunnel (e.g., `HomeLabTunnel`)
5. Choose environment as `Docker` if you are using Docker, or `Linux` if you are not using Docker.

_NOTE_: Pause this activity to go and setup the Cloudflare service on your server. You can refer to the [Setup Cloudflare Service on your Server](#setup-cloudflare-service-on-your-server) section below.

6. Click `Next` once you have setup the Cloudflare service on your server. Now you will go to the `Public Hostname` section.
7. In the `Public Hostname` section, enter the hostname you want to use for your tunnel (e.g., `tunnel.yourdomain.com`) and configure the local `http` address in your local network (e.g., `http://localhost:8080`)
8. You can configure for all the services here like `immich.yourdomain.in` or `authentik.yourdomain.in` or any other service you want to expose through the Cloudflare tunnel.
9. Click `Next` to proceed to the `Private Network` section. You can skip this section if you don't need to connect to a private network.
10. Click `Save` to create the tunnel and you should see the status as `Healthy` once the tunnel is established successfully.

# Setup Cloudflare Service on your Server
1. Go to your server where you want to run the Cloudflare tunnel. Using `Portainer`, create a `stack` or `container` with the following configuration:

```yaml
services:
  cloudflared:
    image: cloudflare/cloudflared
    container_name: cloudflared
    environment:
      - TZ=America/New_York # Change this to your timezone
      - TUNNEL_TOKEN=${TOKEN}
    restart: unless-stopped
    command: tunnel --no-autoupdate run
    networks:
      - cloudflared

networks:
  cloudflared:
    name: cloudflared
```

2. Create a `.env` file in the same directory as your `docker-compose.yml` file with the following
```ini
TOKEN=your_tunnel_token_here
```
3. Replace `your_tunnel_token_here` with the token you received when you created the tunnel in the Cloudflare dashboard.
4. Deploy the stack or container in Portainer.
5. Once the container is running, you can check the logs to ensure that the tunnel is established successfully or on the Cloudflare dashboard under the `Tunnels` section to see the status of your tunnel.

# Test the Clouflare Tunnel Connections
1. Simply open a web browser and navigate to the hostname you configured in the Cloudflare tunnel (e.g., `https://immich.yourdomain.com` or `authentik.yourdomain.com`).
2. You should be able to access your services securely through the Cloudflare tunnel without exposing your server's IP address directly to the internet.

# Configure Securtity Settings using Application and Policies
1. In the Cloudflare dashboard, go to the "Zero Trust" tab and click on `Access` -> `Applications`.
2. Click on `Add an application` and select `Self-hosted`.
3. Enter the name of you application (e.g., `homelab`) and the hostname you configured in the Cloudflare tunnel (e.g., `immich.yourdomain.com`).
4. Under `Policies`, you can configure access policies to restrict who can access your application. You can use email-based policies, IP-based policies, or any other policy that suits your needs.
    - Under `Access` -> `Policies`, you can create a policy.
    - Simply use policies to restrict access to only specific countries and IP addresses.
5. Under `Login methods`, you can configure the login method you want to use for your application. You can use Cloudflare Access, Google, GitHub, or any other identity provider that Cloudflare supports.
    - Default is `One-time PIN` which is a good option for personal use.
    - You can skip other options as we will probably use `Authentik` as our Identity Provider, unless you want to use other methods.
6. Click `Save` to create the application and you should see the status as `Healthy` once the application is created successfully.

# Test the Cloudflare Access Application
1. Based on your configured `policies`, you will be prompted to log in using the configured login method when you try to access the application.
2. If you have configured the policies to restrict access to specific countries, you can test the access by trying to access the application from different countries or IP addresses.