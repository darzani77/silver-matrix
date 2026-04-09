# Silver Matrix 🥈🔗

Ein umfassendes Projekt zur Erstellung eines **Matrix**-Servers, der die Integration mit allen großen Social-Media-Plattformen (**Telegram, Discord, Signal, WhatsApp**) unterstützt und vollständige Audio- und Videoanrufe über **LiveKit** bietet.

Dieses Projekt ist so konzipiert, dass es einfach mit **Docker** installiert werden kann, einschließlich eines modernen **OIDC**-Authentifizierungssystems (MAS) und eines umfassenden Installationsleitfadens.

## Hauptfunktionen 🚀

- **Matrix Synapse**: Das Herzstück des Servers.
- **Bridges**: Unterstützung für (WhatsApp, Telegram, Discord, Signal).
- **Anrufe & Video**: Unterstützung für Gruppen-Video- und Audioanrufe über LiveKit.
- **Authentifizierung**: Modernes Authentifizierungssystem (MAS) mit OIDC-Unterstützung.
- **Datenbank**: Verwendung von PostgreSQL zur effizienten Datenverwaltung.
- **Element Web**: Eine webbasierte Benutzeroberfläche für Matrix.
- **Dockerized**: Führen Sie das gesamte Projekt mit einem Klick über Docker Compose aus.

---

## Projektstruktur 📁

- `synapse/`: Matrix-Server-Konfigurationen.
- `mas/`: Konfigurationen für den Matrix Authentication Service.
- `bridges/`: Konfigurationen für die verschiedenen Bridges (Discord, Signal, WhatsApp, Telegram).
- `livekit/`: Konfigurationen für den Anrufserver.
- `element/`: Konfigurationen für die Web-Oberfläche.
- `data/`: Persistente Daten für die Datenbank.
- `docker-compose.yml`: Die Datei, die alle Dienste ausführt.
- `.env.example`: Eine Beispieldatei für Umgebungsvariablen.

---

## Systemanforderungen 🛠️

1. Ein Linux-Server (VPS) (Ubuntu 22.04+ empfohlen).
2. Installiertes **Docker** und **Docker Compose**.
3. Ein Domainname mit A-Records, die auf die IP-Adresse des Servers verweisen (z.B. `yourdomain.com`, `auth.yourdomain.com`, `element.yourdomain.com`, `rtc.yourdomain.com`, `jwt.yourdomain.com`).
4. **WICHTIG**: Offene Ports in Ihrer Firewall für die Dienste:
   - `80` (HTTP für Element Web)
   - `443` (HTTPS, falls Sie einen externen Reverse Proxy verwenden)
   - `8008` (Synapse Client API)
   - `8448` (Synapse Federation API)
   - `8090` (MAS)
   - `7880`, `7881` (LiveKit TCP)
   - `3478/udp`, `5349/tcp` (LiveKit TURN/STUN)
   - `50100-50200/udp` (LiveKit WebRTC Media)

---

## Installations- und Ausführungsanleitung ⚙️

### 1. Vorbereiten der Dateien
Laden Sie das Projekt herunter, entpacken Sie es und navigieren Sie in den Ordner:
```bash
cd silver-matrix
```

Erstellen Sie eine `.env`-Datei aus der `.env.example`-Datei:
```bash
cp .env.example .env
```

### 2. Anpassen der Konfigurationsdateien
Sie müssen die folgenden Dateien durchgehen und alle Platzhalter ersetzen. Die Platzhalter sind deutlich markiert:
- **`.env`**: Bearbeiten Sie diese Datei, um Ihre Domain, Datenbank-Passwörter, LiveKit API-Schlüssel und den TURN-Geheimschlüssel festzulegen.
  - `DOMAIN`: Ersetzen Sie dies durch Ihren Hauptdomainnamen (z. B. `example.com`).
  - `DB_USER`, `DB_PASSWORD`, `DB_NAME`: Legen Sie die Datenbankzugangsdaten fest.
  - `LIVEKIT_API_KEY` und `LIVEKIT_API_SECRET`: Generieren Sie diese Schlüssel für LiveKit.
  - `TURN_SECRET`: Generieren Sie einen starken, zufälligen Geheimschlüssel für den TURN-Server.
  - `TZ`, `PUID`, `PGID`: Passen Sie diese bei Bedarf an Ihre Systemkonfiguration an.

- **`synapse/homeserver.yaml`**:
  - `YOUR_DOMAIN`: Ersetzen Sie dies durch Ihren Domainnamen.
  - `REPLACE_WITH_MAS_SECRET`, `REPLACE_WITH_DB_PASSWORD`, `REPLACE_WITH_REGISTRATION_SECRET`, `REPLACE_WITH_MACAROON_SECRET`, `REPLACE_WITH_FORM_SECRET`: Ersetzen Sie diese durch starke, zufällig generierte Schlüssel.
  - `REPLACE_WITH_TURN_SECRET`: Ersetzen Sie dies durch den Geheimschlüssel, den Sie in der `.env`-Datei festgelegt haben. Dies ist entscheidend für die Funktionalität von Anrufen auf iOS-Geräten.

- **`mas/config.yaml`**:
  - `YOUR_DOMAIN`: Ersetzen Sie dies durch Ihren Domainnamen.
  - `REPLACE_WITH_OIDC_CLIENT_SECRET`, `REPLACE_WITH_AUTHENTIK_CLIENT_ID`, `REPLACE_WITH_AUTHENTIK_CLIENT_SECRET`, `REPLACE_WITH_EMAIL`, `REPLACE_WITH_SMTP_PASSWORD`, `REPLACE_WITH_ENCRYPTION_SECRET`: Ersetzen Sie diese durch entsprechende Werte.
  - **WICHTIG**: Generieren Sie Ihre eigenen RSA- und EC-Privatschlüssel und ersetzen Sie die Platzhalter in diesem Abschnitt. Anweisungen zur Generierung finden Sie in der Datei.

- **`bridges/discord/config.yaml`**, **`bridges/signal/config.yaml`**, **`bridges/whatsapp/config.yaml`**, **`bridges/telegram/config.yaml`**:
  - `YOUR_DOMAIN`: Ersetzen Sie dies durch Ihren Domainnamen.
  - `REPLACE_WITH_DB_PASSWORD`, `REPLACE_WITH_AS_TOKEN`, `REPLACE_WITH_HS_TOKEN`, `REPLACE_WITH_AVATAR_PROXY_KEY`, `REPLACE_WITH_PROVISIONING_SECRET`, `REPLACE_WITH_SIGNING_KEY`, `REPLACE_WITH_PICKLE_KEY`: Ersetzen Sie diese durch starke, zufällig generierte Schlüssel.
  - `REPLACE_WITH_ADMIN_MXID`: Ersetzen Sie dies durch die Matrix-ID Ihres Admin-Benutzers (z.B. `@admin:yourdomain.com`).

- **`livekit/config.yaml`**:
  - `YOUR_DOMAIN`: Ersetzen Sie dies durch Ihren Domainnamen.
  - `REPLACE_WITH_LIVEKIT_API_KEY`, `REPLACE_WITH_LIVEKIT_API_SECRET`: Ersetzen Sie diese durch die Schlüssel, die Sie in der `.env`-Datei festgelegt haben.
  - **WICHTIG**: Dieser Datei enthält nun die Konfiguration für den integrierten TURN-Server, der für iOS-Anrufe unerlässlich ist.

- **`element/config.json`**:
  - `YOUR_DOMAIN`: Ersetzen Sie dies durch Ihren Domainnamen.

> **Tipp zur Schlüsselgenerierung**: Sie können den Befehl `openssl rand -hex 32` verwenden, um sichere Zufallsschlüssel zu generieren.

### 3. Generieren der Synapse-Signaturschlüssel (Signing Keys)
Sie müssen den Signaturschlüssel für Synapse generieren, bevor Sie den Server starten. Ersetzen Sie `YOUR_DOMAIN` durch Ihre tatsächliche Domain:
```bash
docker run -it --rm -v $(pwd)/synapse:/data matrixdotorg/synapse:latest generate-config --report-stats=no --server-name YOUR_DOMAIN
```

### 4. Starten des Servers
Sobald die Konfiguration abgeschlossen ist, starten Sie alle Dienste:
```bash
docker compose up -d
```

---

## Verbinden der Bridges 🔗

Nach dem Start des Servers müssen Sie die Registrierungsdateien (`registration.yaml`) für jede Bridge generieren, damit Synapse mit ihnen kommunizieren kann. Diese Dateien müssen dann in den `synapse/` Ordner kopiert und in der `homeserver.yaml` referenziert werden.

**Beispiel für WhatsApp:**
1.  Generieren Sie die Registrierungsdatei:
    ```bash
    docker compose exec whatsapp-bridge python -m mautrix_whatsapp -g -c /data/config.yaml -r /data/registration.yaml
    ```
2.  Kopieren Sie die generierte `registration.yaml` in den `synapse/` Ordner:
    ```bash
    cp bridges/whatsapp/registration.yaml synapse/whatsapp-registration.yaml
    ```
3.  Stellen Sie sicher, dass `synapse/homeserver.yaml` die Zeile `- /data/whatsapp-registration.yaml` unter `app_service_config_files:` enthält.
4.  Starten Sie Synapse neu, damit die Änderungen wirksam werden:
    ```bash
    docker compose restart synapse
    ```

**Wiederholen Sie diesen Vorgang für jede Bridge (Telegram, Signal, Discord).**

Danach können Sie die Bridges aktivieren, indem Sie Nachrichten an ihre Bots in Matrix senden:
- **WhatsApp**: Senden Sie `login` an den WhatsApp-Bot und scannen Sie den QR-Code.
- **Telegram**: Senden Sie `login` an den Telegram-Bot und folgen Sie den Anweisungen.
- **Discord**: Erfordert zuerst die Einrichtung eines Discord-Bot-Tokens in der Konfigurationsdatei und dann das Senden von `login` an den Discord-Bot.
- **Signal**: Erfordert die Verknüpfung Ihres Signal-Kontos über den Bot.

---

## Fehlerbehebung: Audio auf iOS-Geräten 🔊

Wenn Sie Probleme mit dem Audio bei Anrufen auf iOS-Geräten haben, stellen Sie sicher, dass:
- Der integrierte TURN-Server in `livekit/config.yaml` aktiviert ist.
- Die `turn_uris` und `turn_shared_secret` in `synapse/homeserver.yaml` korrekt konfiguriert sind.
- Die Ports `3478/udp` und `5349/tcp` für LiveKit in Ihrer Firewall geöffnet sind.
- Die Domain `rtc.YOUR_DOMAIN` korrekt auf die IP-Adresse Ihres Servers verweist.

---

## Support und Beitrag 🤝
Dieses Projekt ist Open Source und für alle verfügbar. Wenn Sie Verbesserungen oder Fehlerbehebungen hinzufügen, senden Sie bitte einen **Pull Request**, um anderen zu helfen.

---

**Erstellt von iDrzn - Entwickler des Silver Matrix Projekts**
