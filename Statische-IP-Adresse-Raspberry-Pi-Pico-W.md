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

## Werte anpassen

| Parameter | Beispiel | Beschreibung |
|-----------|----------|--------------|
| IP | `192.168.1.100` | Gewuenschte feste IP (frei im Netzwerk) |
| Subnetzmaske | `255.255.255.0` | Meistens so belassen |
| GATEWAY | `192.168.1.1` | IP deines Routers |
| DNS | `8.8.8.8` | Google DNS oder Router-IP |

---

## So verwendest du es:

  1. Öffne die Datei und passe diese Werte an: WLAN_SSID, WLAN_PASSWORT, STATISCHE_IP, GATEWAY - IP
  2. Kopiere die Datei auf den Pico W (z.B. mit Thonny)
  3. Der Pico W startet automatisch mit main.py

## Tipps

---

- Die IP-Adresse im DHCP-Bereich des Routers **ausschliessen** oder eine Adresse ausserhalb des DHCP-Bereichs waehlen, damit es keine Konflikte gibt
- Den Code in `main.py` auf dem Pico W speichern, damit er automatisch beim Booten laeuft
- Gateway-Adresse findest du auf deinem Router oder per `ipconfig` (Windows) / `ip route` (Linux)
