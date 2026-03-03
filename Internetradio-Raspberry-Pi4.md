# Internetradio auf dem Raspberry Pi 4 einrichten

## Voraussetzungen

- Raspberry Pi 4 mit Raspberry Pi OS
- Netzwerkverbindung (LAN oder WLAN)
- Lautsprecher (ueber 3,5mm Klinke, HDMI oder USB-DAC)
- Optional: Display, Tasten/Drehregler fuer Bedienung

---

## Variante 1: MPD + MPC (leichtgewichtig, Terminal-basiert)

### Installation

```bash
sudo apt update
sudo apt install mpd mpc
```

### MPD konfigurieren

```bash
sudo nano /etc/mpd.conf
```

Wichtige Einstellungen:

```
music_directory    "/var/lib/mpd/music"
playlist_directory "/var/lib/mpd/playlists"

audio_output {
    type    "alsa"
    name    "My ALSA Device"
    mixer_type "software"
}
```

MPD neu starten:

```bash
sudo systemctl restart mpd
sudo systemctl enable mpd
```

### Radiosender hinzufuegen

Playlist-Datei erstellen:

```bash
sudo nano /var/lib/mpd/playlists/radio.m3u
```

Beispielsender eintragen:

```
#EXTM3U

#EXTINF:-1,SWR3
https://liveradio.swr.de/sw282p3/swr3/play.mp3

#EXTINF:-1,WDR 2
https://wdr-wdr2-rheinland.icecastssl.wdr.de/wdr/wdr2/rheinland/mp3/128/stream.mp3

#EXTINF:-1,Bayern 3
https://streams.br.de/bayern3_2.m3u

#EXTINF:-1,Radio Bob
https://streams.radiobob.de/bob-live/mp3-192/mediaplayer
```

### Bedienung ueber MPC

```bash
# Playlist laden
mpc load radio

# Abspielen
mpc play

# Naechster Sender
mpc next

# Vorheriger Sender
mpc prev

# Lautstaerke setzen (0-100)
mpc volume 80

# Stoppen
mpc stop

# Aktuellen Sender anzeigen
mpc current
```

---

## Variante 2: VLC (einfach, vielseitig)

### Installation

```bash
sudo apt update
sudo apt install vlc
```

### Sender abspielen

```bash
# Einzelnen Stream abspielen
cvlc https://liveradio.swr.de/sw282p3/swr3/play.mp3

# Mit Lautstaerke (0.0 - 1.0)
cvlc --gain 0.8 https://liveradio.swr.de/sw282p3/swr3/play.mp3
```

### Playlist erstellen

```bash
nano ~/radio-playlist.m3u
```

```
#EXTM3U
#EXTINF:-1,SWR3
https://liveradio.swr.de/sw282p3/swr3/play.mp3
#EXTINF:-1,WDR 2
https://wdr-wdr2-rheinland.icecastssl.wdr.de/wdr/wdr2/rheinland/mp3/128/stream.mp3
#EXTINF:-1,Bayern 3
https://streams.br.de/bayern3_2.m3u
```

```bash
cvlc ~/radio-playlist.m3u
```

---

## Variante 3: Mopidy (Web-Oberflaeche)

### Installation

```bash
# Mopidy und Erweiterungen installieren
sudo apt update
sudo apt install mopidy mopidy-mpd

# Web-Frontend installieren
sudo python3 -m pip install Mopidy-Iris
```

### Konfiguration

```bash
sudo nano /etc/mopidy/mopidy.conf
```

```ini
[core]
restore_state = true

[audio]
output = autoaudiosink

[http]
hostname = 0.0.0.0
port = 6680

[mpd]
hostname = 0.0.0.0
port = 6600

[stream]
enabled = true
```

Mopidy starten:

```bash
sudo systemctl enable mopidy
sudo systemctl start mopidy
```

### Web-Oberflaeche aufrufen

Im Browser oeffnen:

```
http://<raspberry-pi-ip>:6680/iris/
```

Dort koennen Streams direkt eingefuegt und abgespielt werden.

---

## Audio-Ausgabe konfigurieren

### Ausgabegeraet pruefen

```bash
aplay -l
```

### Ausgabe umschalten

```bash
# 3,5mm Klinke erzwingen
sudo raspi-config
# → System Options → Audio → Headphones

# Oder per amixer
amixer cset numid=3 1   # 1=Klinke, 2=HDMI, 0=Auto
```

### Lautstaerke einstellen

```bash
alsamixer
```

Oder direkt:

```bash
amixer set Master 80%
```

---

## USB-Audio-Ausgabe (USB-DAC / USB-Soundkarte)

Die meisten USB-Audiogeraete werden vom Pi ohne zusaetzliche Treiber erkannt (Plug & Play).

### 1. USB-Soundkarte anschliessen und erkennen

```bash
# Angeschlossene Audiogeraete anzeigen
aplay -l
```

Ausgabe z.B.:

```
**** List of PLAYBACK Hardware Devices ****
card 0: Headphones [bcm2835 Headphones], device 0: ...
card 1: Device [USB Audio Device], device 0: ...
```

Hier ist `card 1` die USB-Soundkarte.

### 2. USB-Audio als Standard setzen

```bash
sudo nano /usr/share/alsa/alsa.conf
```

Zeilen aendern:

```
defaults.ctl.card 1
defaults.pcm.card 1
```

Oder alternativ per Benutzerkonfiguration:

```bash
nano ~/.asoundrc
```

```
pcm.!default {
    type hw
    card 1
}

ctl.!default {
    type hw
    card 1
}
```

### 3. MPD fuer USB-Audio konfigurieren

In `/etc/mpd.conf`:

```
audio_output {
    type    "alsa"
    name    "USB DAC"
    device  "hw:1,0"
    mixer_type "software"
}
```

Danach:

```bash
sudo systemctl restart mpd
```

### 4. Testen

```bash
# Testton ueber USB abspielen
speaker-test -D hw:1,0 -c 2 -t wav

# Lautstaerke regeln
amixer -c 1 set Speaker 80%
```

### Empfehlenswerte USB-DACs

| Geraet | Preis ca. | Bemerkung |
|--------|-----------|-----------|
| USB-Soundkarte (generisch) | 5-10 EUR | Einfach, funktioniert sofort |
| Sabrent USB Audio Adapter | 8 EUR | Klinke + Mikrofon |
| FiiO E10K | 80 EUR | HiFi-Qualitaet |
| HifiBerry DAC+ (HAT) | 35 EUR | Kein USB, direkt auf GPIO |

---

## Autostart: Radio beim Booten abspielen

### Systemd-Service erstellen

```bash
sudo nano /etc/systemd/system/internetradio.service
```

```ini
[Unit]
Description=Internetradio
After=network-online.target sound.target
Wants=network-online.target

[Service]
Type=simple
User=pi
ExecStartPre=/bin/sleep 5
ExecStart=/usr/bin/mpc play
Restart=on-failure
RestartSec=10

[Install]
WantedBy=multi-user.target
```

Service aktivieren:

```bash
sudo systemctl daemon-reload
sudo systemctl enable internetradio.service
```

---

## Optional: Hardware-Tasten fuer Bedienung

### GPIO-Tasten mit Python

```bash
sudo apt install python3-gpiozero
```

Skript erstellen:

```bash
nano ~/radio-buttons.py
```

```python
#!/usr/bin/env python3
from gpiozero import Button
import subprocess

btn_play = Button(17)    # GPIO 17 - Play/Pause
btn_next = Button(27)    # GPIO 27 - Naechster Sender
btn_prev = Button(22)    # GPIO 22 - Vorheriger Sender
btn_vol_up = Button(23)  # GPIO 23 - Lauter
btn_vol_dn = Button(24)  # GPIO 24 - Leiser

def mpc(cmd):
    subprocess.run(["mpc"] + cmd.split())

btn_play.when_pressed = lambda: mpc("toggle")
btn_next.when_pressed = lambda: mpc("next")
btn_prev.when_pressed = lambda: mpc("prev")
btn_vol_up.when_pressed = lambda: mpc("volume +5")
btn_vol_dn.when_pressed = lambda: mpc("volume -5")

print("Radio-Tasten aktiv...")
from signal import pause
pause()
```

```bash
chmod +x ~/radio-buttons.py
python3 ~/radio-buttons.py
```

---

## Nuetzliche Radio-Stream-Verzeichnisse

| Quelle | URL |
|--------|-----|
| Radio-Browser API | https://www.radio-browser.info |
| SWR Streams | https://www.swr.de/unternehmen/empfang/livestreams-100.html |
| Community Radio Browser | https://www.radio-browser.info/search |

---

## Troubleshooting

| Problem | Loesung |
|---------|---------|
| Kein Ton | `alsamixer` pruefen, Lautstaerke hochdrehen |
| Stream bricht ab | Netzwerk pruefen, `mpc repeat on` setzen |
| MPD startet nicht | `sudo systemctl status mpd` pruefen |
| Falsche Ausgabe | `raspi-config` → Audio-Ausgabe waehlen |
