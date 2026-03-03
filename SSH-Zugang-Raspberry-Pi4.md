# SSH-Zugang auf dem Raspberry Pi 4 einrichten

## Variante 1: SSH ueber raspi-config aktivieren (mit Monitor)

```bash
sudo raspi-config
```

Navigiere zu: **Interface Options → SSH → Yes**

Danach neustarten:

```bash
sudo reboot
```

---

## Variante 2: SSH headless aktivieren (ohne Monitor)

SD-Karte am PC einlegen und im **boot**-Verzeichnis (bei Bookworm: **bootfs**) eine leere Datei namens `ssh` erstellen:

```bash
# Linux/Mac
touch /Volumes/bootfs/ssh

# Windows (PowerShell)
New-Item -Path "D:\ssh" -ItemType File
```

Beim naechsten Start ist SSH automatisch aktiv.

---

## Variante 3: Ueber den Raspberry Pi Imager

Beim Flashen des OS im **Raspberry Pi Imager**:
1. Zahnrad-Symbol klicken (oder `Ctrl+Shift+X`)
2. **SSH aktivieren** anhaken
3. Benutzername und Passwort festlegen
4. Image schreiben

---

## Verbindung herstellen

### IP-Adresse des Pi ermitteln

Am Pi selbst:
```bash
hostname -I
```

Oder vom PC aus das Netzwerk scannen:
```bash
# Linux/Mac
ping raspberrypi.local

# Oder mit nmap
nmap -sn 192.168.1.0/24
```

### Per SSH verbinden

```bash
ssh benutzername@192.168.1.100
```

Beim ersten Verbinden den Fingerprint mit `yes` bestaetigen.

---

## SSH absichern

### 1. SSH-Key erstellen (auf dem PC)

```bash
ssh-keygen -t ed25519 -C "mein-pc"
```

### 2. Public Key auf den Pi kopieren

```bash
ssh-copy-id benutzername@192.168.1.100
```

### 3. Passwort-Login deaktivieren

Auf dem Pi:

```bash
sudo nano /etc/ssh/sshd_config
```

Folgende Zeilen setzen:

```
PasswordAuthentication no
PermitRootLogin no
```

SSH-Dienst neu starten:

```bash
sudo systemctl restart ssh
```

### 4. SSH-Port aendern (optional)

In `/etc/ssh/sshd_config`:

```
Port 2222
```

Danach verbinden mit:

```bash
ssh -p 2222 benutzername@192.168.1.100
```

---

## SSH-Status pruefen

```bash
# Ist SSH aktiv?
sudo systemctl status ssh

# SSH aktivieren/starten
sudo systemctl enable ssh
sudo systemctl start ssh
```

---

## Wichtige Hinweise

- **Standardport** ist `22` — in oeffentlichen Netzen sollte dieser geaendert werden
- **Root-Login** sollte immer deaktiviert bleiben
- **SSH-Keys** sind sicherer als Passwoerter und sollten bevorzugt werden
- **Fail2ban** installieren, um Brute-Force-Angriffe zu blockieren:
  ```bash
  sudo apt install fail2ban
  ```
