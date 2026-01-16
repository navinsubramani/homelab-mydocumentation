# Table of Contents
- [Install Home Assistant on Server](#installing-homeassistant)
- [Setup Home Assistant App on Mobile](#setup-homeassistant)
- [Configure the Two-Factor Authentication (2FA)](#setup-2fa-homeassistant)
- [Configure the Home Assistant for Remote Access](#setup-remote-access-homeassistant)
- [Configure the Home Assistant Alarm](#setup-alarm-homeassistant)
- [Configuring the Home Assistant to edit the configuration.yaml file via the UI](#setup-edit-configuration-homeassistant)
- [Integrating Amcrest Camera to Home Assistant](#setup-amcrest-homeassistant)
- [Configure Home Assistant Backup](#setup-backup-homeassistant)

![Home Assistant](../static/Homeassistant_hardwares.png)

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
    - Primarily the steps involve adding the following lines to your `configuration.yaml` file.
3. Restart the Home Assistant instance to apply the changes.

# Configure the Home Assistant Alarm <a name="setup-alarm-homeassistant"></a>

1. Reference: [ios critical alerts](https://companion.home-assistant.io/docs/notifications/critical-notifications/)

Below is the reference yaml code to configure a critical alert for Home Assistant Alarm.

```yaml
alias: Motion Detection
description: ""
triggers:
  - type: motion
    device_id: <idevice_id>
    entity_id: <entity_id>
    domain: binary_sensor
    trigger: device
    for:
      hours: 0
      minutes: 0
      seconds: 0
actions:
  - action: notify.mobile_app_<your_device_name>
    data:
      title: Motion Detected in Living Room
      message: There is a motion detected in the living room
      data:
        push:
          sound:
            name: default
            critical: 1
            volume: 1
mode: single
```

# Configuring the Home Assistant to edit the configuration.yaml file via the UI <a name="setup-edit-configuration-homeassistant"></a>
1. In Home Assistant, navigate to `Settings` > `Add-ons` > `Add-on Store`.
2. Search for `File Editor` and click on it.
3. Click on `Install` to add the File Editor add-on to your Home Assistant instance.
4. Once installed, start the add-on and optionally enable `Show in sidebar` for easy access.
5. You can now access the File Editor from the sidebar and edit the `configuration.yaml` file directly through the Home Assistant UI. Remember to check the configuration for errors and restart Home Assistant after making changes.

# Integrating Amcrest Camera to Home Assistant <a name="setup-amcrest-homeassistant"></a>

Follow the official documentation to integrate Amcrest Camera with Home Assistant [here](https://www.home-assistant.io/integrations/amcrest/).


# Configure Home Assistant Backup <a name="setup-backup-homeassistant"></a>
1. In Home Assistant, navigate to `Settings` > `System` > `Backups`.
2. Click on `New Backup Now` to create a new backup of your Home Assistant configuration.
3. You can download the backup file to your local machine for safekeeping by clicking on the backup and selecting `Download`.
