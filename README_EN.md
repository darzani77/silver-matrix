# Silver Matrix 🥈🔗

A comprehensive project for setting up a **Matrix** server that supports integration with all major social media platforms (**Telegram, Discord, Signal, WhatsApp**) and offers full audio and video calling via **LiveKit**.

This project is designed to be easily installed using **Docker**, including a modern **OIDC** authentication system (MAS) and a comprehensive installation guide.

## Key Features 🚀

- **Matrix Synapse**: The core of the server.
- **Bridges**: Support for (WhatsApp, Telegram, Discord, Signal).
- **Calls & Video**: Support for group video and audio calls via LiveKit, optimized for iOS.
- **Authentication**: Modern Authentication System (MAS) with OIDC support.
- **Database**: Utilizing PostgreSQL for efficient data management.
- **Element Web**: A web-based user interface for Matrix.
- **Dockerized**: Run the entire project with a single click via Docker Compose.

---

## Project Structure 📁

- `synapse/`: Matrix server configurations.
- `mas/`: Configurations for the Matrix Authentication Service.
- `bridges/`: Configurations for the various bridges (Discord, Signal, WhatsApp, Telegram).
- `livekit/`: Configurations for the calling server.
- `element/`: Configurations for the web interface.
- `data/`: Persistent data for the database.
- `docker-compose.yml`: The file that runs all services.
- `.env.example`: A sample file for environment variables.

---

## System Requirements 🛠️

1. A Linux server (VPS) (Ubuntu 22.04+ recommended).
2. Installed **Docker** and **Docker Compose**.
3. A domain name with A-records pointing to the server's IP address (e.g., `yourdomain.com`, `auth.yourdomain.com`, `element.yourdomain.com`, `rtc.yourdomain.com`, `jwt.yourdomain.com`).
4. **IMPORTANT**: Open ports in your firewall for the services:
   - `80` (HTTP for Element Web)
   - `443` (HTTPS, if you use an external Reverse Proxy)
   - `8008` (Synapse Client API)
   - `8448` (Synapse Federation API)
   - `8090` (MAS)
   - `7880`, `7881` (LiveKit TCP)
   - `3478/udp`, `5349/tcp` (LiveKit TURN/STUN)
   - `50100-50200/udp` (LiveKit WebRTC Media)

---

## Installation and Execution Guide ⚙️

### 1. Preparing the Files
Download the project, extract it, and navigate into the folder:
```bash
cd silver-matrix
```

Create an `.env` file from the `.env.example` file:
```bash
cp .env.example .env
```

### 2. Customizing Configuration Files
You must go through the following files and replace all placeholders. The placeholders are clearly marked:
- **`.env`**: Edit this file to set your domain, database passwords, LiveKit API keys, and the TURN secret.
  - `DOMAIN`: Replace this with your main domain name (e.g., `example.com`).
  - `DB_USER`, `DB_PASSWORD`, `DB_NAME`: Set the database credentials.
  - `LIVEKIT_API_KEY` and `LIVEKIT_API_SECRET`: Generate these keys for LiveKit.
  - `TURN_SECRET`: Generate a strong, random secret for the TURN server.
  - `TZ`, `PUID`, `PGID`: Adjust these as needed for your system configuration.

- **`synapse/homeserver.yaml`**:
  - `YOUR_DOMAIN`: Replace this with your domain name.
  - `REPLACE_WITH_MAS_SECRET`, `REPLACE_WITH_DB_PASSWORD`, `REPLACE_WITH_REGISTRATION_SECRET`, `REPLACE_WITH_MACAROON_SECRET`, `REPLACE_WITH_FORM_SECRET`: Replace these with strong, randomly generated keys.
  - `REPLACE_WITH_TURN_SECRET`: Replace this with the secret you set in the `.env` file. This is crucial for iOS call functionality.

- **`mas/config.yaml`**:
  - `YOUR_DOMAIN`: Replace this with your domain name.
  - `REPLACE_WITH_OIDC_CLIENT_SECRET`, `REPLACE_WITH_AUTHENTIK_CLIENT_ID`, `REPLACE_WITH_AUTHENTIK_CLIENT_SECRET`, `REPLACE_WITH_EMAIL`, `REPLACE_WITH_SMTP_PASSWORD`, `REPLACE_WITH_ENCRYPTION_SECRET`: Replace these with appropriate values.
  - **IMPORTANT**: Generate your own RSA and EC private keys and replace the placeholders in this section. Instructions for generation are provided within the file.

- **`bridges/discord/config.yaml`**, **`bridges/signal/config.yaml`**, **`bridges/whatsapp/config.yaml`**, **`bridges/telegram/config.yaml`**:
  - `YOUR_DOMAIN`: Replace this with your domain name.
  - `REPLACE_WITH_DB_PASSWORD`, `REPLACE_WITH_AS_TOKEN`, `REPLACE_WITH_HS_TOKEN`, `REPLACE_WITH_AVATAR_PROXY_KEY`, `REPLACE_WITH_PROVISIONING_SECRET`, `REPLACE_WITH_SIGNING_KEY`, `REPLACE_WITH_PICKLE_KEY`: Replace these with strong, randomly generated keys.
  - `REPLACE_WITH_ADMIN_MXID`: Replace this with the Matrix ID of your admin user (e.g., `@admin:yourdomain.com`).

- **`livekit/config.yaml`**:
  - `YOUR_DOMAIN`: Replace this with your domain name.
  - `REPLACE_WITH_LIVEKIT_API_KEY`, `REPLACE_WITH_LIVEKIT_API_SECRET`: Replace these with the keys you set in the `.env` file.
  - **IMPORTANT**: This file now contains the configuration for the integrated TURN server, which is essential for iOS calls.

- **`element/config.json`**:
  - `YOUR_DOMAIN`: Replace this with your domain name.

> **Key Generation Tip**: You can use the command `openssl rand -hex 32` to generate secure random keys.

### 3. Generating Synapse Signing Keys
You must generate the signing key for Synapse before starting the server. Replace `YOUR_DOMAIN` with your actual domain:
```bash
docker run -it --rm -v $(pwd)/synapse:/data matrixdotorg/synapse:latest generate-config --report-stats=no --server-name YOUR_DOMAIN
```

### 4. Starting the Server
Once the configuration is complete, start all services:
```bash
docker compose up -d
```

---

## Connecting the Bridges 🔗

After starting the server, you need to generate the registration files (`registration.yaml`) for each bridge so Synapse can communicate with them. These files must then be copied into the `synapse/` folder and referenced in the `homeserver.yaml`.

**Example for WhatsApp:**
1.  Generate the registration file:
    ```bash
    docker compose exec whatsapp-bridge python -m mautrix_whatsapp -g -c /data/config.yaml -r /data/registration.yaml
    ```
2.  Copy the generated `registration.yaml` into the `synapse/` folder:
    ```bash
    cp bridges/whatsapp/registration.yaml synapse/whatsapp-registration.yaml
    ```
3.  Ensure `synapse/homeserver.yaml` contains the line `- /data/whatsapp-registration.yaml` under `app_service_config_files:`.
4.  Restart Synapse for the changes to take effect:
    ```bash
    docker compose restart synapse
    ```

**Repeat this process for each bridge (Telegram, Signal, Discord).**

After that, you can activate the bridges by sending messages to their bots in Matrix:
- **WhatsApp**: Send `login` to the WhatsApp bot and scan the QR code.
- **Telegram**: Send `login` to the Telegram bot and follow the instructions.
- **Discord**: Requires setting up a Discord Bot Token in the configuration file first and then sending `login` to the Discord bot.
- **Signal**: Requires linking your Signal account via the bot.

---

## Troubleshooting: Audio on iOS Devices 🔊

If you experience audio issues during calls on iOS devices, ensure the following:
- The integrated TURN server in `livekit/config.yaml` is enabled and configured correctly.
- The `turn_uris` and `turn_shared_secret` in `synapse/homeserver.yaml` are correctly configured.
- Ports `3478/udp` and `5349/tcp` for LiveKit are open in your firewall.
- The domain `rtc.YOUR_DOMAIN` correctly points to your server's IP address.

---

## Support and Contribution 🤝
This project is open source and available to everyone. If you add improvements or bug fixes, please submit a **Pull Request** to help others.

---

**Created by iDrzn - Developer of the Silver Matrix Project**
