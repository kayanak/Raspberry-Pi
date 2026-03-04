# Statische IP-Adresse beim Raspberry Pi Pico W einrichten

Beim Pico W wird die statische IP direkt in MicroPython konfiguriert (es gibt kein Linux/raspi-config).

---

## MicroPython-Code

Datei `main.py` auf dem Pico W erstellen:

```python
import network
import time

WLAN_SSID     = "WLAN_NAME"
WLAN_PASSWORT = "WLAN_PASSWORT"

STATISCHE_IP  = "192.168.1.100"      # Gewuenschte IP fuer den Pico W
SUBNETZMASKE  = "255.255.255.0"      # Meist 255.255.255.0
GATEWAY       = "192.168.1.1"        # IP des Routers
DNS_SERVER    = "8.8.8.8"            # DNS (8.8.8.8 = Google DNS)

STATISCHE_IP_VERWENDEN = True

wlan = network.WLAN(network.STA_IF)
wlan.active(True)

# Parameter: (IP, Subnetzmaske, Gateway, DNS)
wlan.ifconfig((STATISCHE_IP, SUBNETZMASKE, GATEWAY, DNS_SERVER))

# WLAN verbinden
print("Verbinde mit WLAN:", WLAN_SSID)
wlan.connect(WLAN_SSID, WLAN_PASSWORT)

# Auf Verbindung warten
while not wlan.isconnected():
    print("Warte auf Verbindung...")
    time.sleep(10)

# Ergebnis pruefen
if wlan.isconnected():
    ip, subnet, gw, dns = wlan.ifconfig()
    print("\n=== Verbindung erfolgreich ===")
    print("IP-Adresse:  ", ip)
    print("Subnetzmaske:", subnet)
    print("Gateway:     ", gw)
    print("DNS:         ", dns)
    return wlan
else:
    print("Verbindung fehlgeschlagen!")
    return None

```

---

## So verwendest du es:

- Öffne die Datei und passe diese Werte an: WLAN_SSID, WLAN_PASSWORT, STATISCHE-IP, GATEWAY-IP
- Kopiere die Datei auf den Pico W (z.B. mit Thonny)
- Der Pico W startet automatisch mit main.py
  
---

## MicroPython auf den Pico W flashen:

Schritt 1: Pico W im Bootloader-Modus starten
- Halte die BOOTSEL-Taste auf dem Pico W gedrückt
- Schließe ihn gleichzeitig per USB an den PC an
- Loslassen – er erscheint als USB-Laufwerk namens RPI-RP2

  Schritt 2: Firmware herunterladen

  - Gehe zu: https://micropython.org/download/RPI_PICO_W/
  - Lade die neueste .uf2-Datei herunter (unter "Releases")

  Schritt 3: Firmware aufspielen

  - Ziehe die .uf2-Datei auf das RPI-RP2 Laufwerk
  - Der Pico startet automatisch neu und verschwindet als Laufwerk
  - Das ist normal – MicroPython läuft jetzt

  Schritt 4: In Thonny verbinden

  - Öffne Thonny
  - Unten rechts auf den Interpreter klicken
  - MicroPython (Raspberry Pi Pico) wählen
  - Unten in der Shell sollte erscheinen: MicroPython v1.xx.x on Raspberry Pi Pico W >>>

  Wenn du das >>> siehst, ist alles bereit und du kannst deine main.py hochladen.

---
