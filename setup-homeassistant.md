# Table of Contents
- [Install Home Assistant on Server](#installing-homeassistant)
- [Setup Home Assistant App on Mobile](#setup-homeassistant)
- [Configure the Two-Factor Authentication (2FA)](#setup-2fa-homeassistant)
- [Configure the Home Assistant for Remote Access](#setup-remote-access-homeassistant)

# Install Home Assistant on Server <a name="installing-homeassistant"></a>
1. Installing `Home Assistant` can be done using their documentation from their [website](https://www.home-assistant.io/installation/raspberrypi). The installation method recommended is using the Home Assistant OS image.
2. Once installed, you can access Home Assistant by navigating to `http://homeassistant.local:8123` in your web browser.
3. Follow the on-screen instructions to complete the initial setup, including creating an account and configuring your home settings.

# Setup Home Assistant App on Mobile <a name="setup-homeassistant"></a>
1. Download the Home Assistant app from the `App Store` for iOS or `Google Play Store` for Android.
2. Open the app and enter the URL of your Home Assistant instance (e.g., `http://homeassistant.local:8123`).
3. Log in using the account you created during the initial setup.
4. You can now start adding integrations and automations to enhance your smart home experience.

# Configure the Two-Factor Authentication (2FA) <a name="setup-2fa-homeassistant"></a>
1. In Home Assistant, navigate to your user profile by clicking on your user icon in the bottom left corner.
2. Scroll down to the "Multi-Factor Authentication" section and click on "Enable".
3. Follow the prompts to set up 2FA using an authenticator app like Google Authenticator or Authy. Scan the QR code provided and enter the verification code to complete the setup.

# Configure the Home Assistant for Remote Access <a name="setup-remote-access-homeassistant"></a>
1. To access the Home Assistant instance remotely, you can use the Cloudflare Tunnel Addon. Refer the [video documentation](https://www.youtube.com/watch?v=XoTmO4mLibw) for detailed steps on setting up Cloudflare Tunnel with Home Assistant.
2. Once the Add-on is setup, you have to also `allow request` from cloudflare addon by enabling the reverse proxies. Follow the steps frmo the official documentation [here](https://github.com/brenner-tobias/addon-cloudflared/blob/main/cloudflared/DOCS.md).
3. Restart the Home Assistant instance to apply the changes.