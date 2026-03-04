# Statische IP-Adresse beim Raspberry Pi Pico W einrichten

Beim Pico W wird die statische IP direkt in MicroPython konfiguriert (es gibt kein Linux/raspi-config).

---

## MicroPython-Code

Datei `main.py` auf dem Pico W erstellen:

```python
import network
import time

 WLAN_SSID = "DEIN_WLAN_NAME"
 WLAN_PASSWORT = "DEIN_PASSWORT"

wlan = network.WLAN(network.STA_IF)
wlan.active(True)

# Statische IP konfigurieren BEVOR die Verbindung hergestellt wird
# Parameter: (IP, Subnetzmaske, Gateway, DNS)
wlan.ifconfig(('192.168.1.100', '255.255.255.0', '192.168.1.1', '8.8.8.8'))

# WLAN verbinden
wlan.connect('DEIN_WLAN_NAME', 'DEIN_WLAN_PASSWORT')

# Auf Verbindung warten
while not wlan.isconnected():
    time.sleep(1)

print('Verbunden mit IP:', wlan.ifconfig()[0])
```

---

## Werte anpassen

| Parameter | Beispiel | Beschreibung |
|-----------|----------|--------------|
| IP | `192.168.1.100` | Gewuenschte feste IP (frei im Netzwerk) |
| Subnetzmaske | `255.255.255.0` | Meistens so belassen |
| Gateway | `192.168.1.1` | IP deines Routers |
| DNS | `8.8.8.8` | Google DNS oder Router-IP |

---

## Tipps

- Die IP-Adresse im DHCP-Bereich des Routers **ausschliessen** oder eine Adresse ausserhalb des DHCP-Bereichs waehlen, damit es keine Konflikte gibt
- Den Code in `main.py` auf dem Pico W speichern, damit er automatisch beim Booten laeuft
- Gateway-Adresse findest du auf deinem Router oder per `ipconfig` (Windows) / `ip route` (Linux)
