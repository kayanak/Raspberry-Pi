# Statische IP-Adresse auf dem Raspberry Pi 4 einrichten

## Voraussetzungen

- Raspberry Pi 4 mit Raspberry Pi OS (Bookworm oder älter)
- Zugang per Terminal (direkt oder SSH)

---

## Schritt 1: Aktuelle Netzwerkkonfiguration ermitteln

```bash
# Aktuelle IP-Adresse anzeigen
ip addr show

# Standard-Gateway ermitteln
ip route show default

# DNS-Server anzeigen
cat /etc/resolv.conf
```

Notiere dir folgende Werte:
- **Aktuelle IP-Adresse** (z.B. `192.168.1.100`)
- **Gateway** (z.B. `192.168.1.1`)
- **DNS-Server** (z.B. `192.168.1.1` oder `8.8.8.8`)
- **Subnetzmaske** als CIDR (meist `/24`)

---

## Schritt 2: Konfiguration je nach OS-Version

### Variante A: Raspberry Pi OS Bookworm (ab 2023) — NetworkManager

Bookworm verwendet **NetworkManager** statt dhcpcd.

**Per Terminal (nmcli):**

```bash
# Verbindungsname herausfinden
nmcli con show

# Statische IP setzen (Beispielwerte anpassen!)
sudo nmcli con mod "Wired connection 1" \
  ipv4.addresses 192.168.1.100/24 \
  ipv4.gateway 192.168.1.1 \
  ipv4.dns "8.8.8.8 8.8.4.4" \
  ipv4.method manual

# Verbindung neu starten
sudo nmcli con up "Wired connection 1"
```

Fuer **WLAN** statt `"Wired connection 1"` den WLAN-Verbindungsnamen verwenden (z.B. `"preconfigured"` oder den SSID-Namen).

---

### Variante B: Raspberry Pi OS Bullseye und aelter — dhcpcd

Die Konfiguration erfolgt ueber `/etc/dhcpcd.conf`:

```bash
sudo nano /etc/dhcpcd.conf
```

Am Ende der Datei folgende Zeilen hinzufuegen:

**Fuer Ethernet (eth0):**

```
interface eth0
static ip_address=192.168.1.100/24
static routers=192.168.1.1
static domain_name_servers=8.8.8.8 8.8.4.4
```

**Fuer WLAN (wlan0):**

```
interface wlan0
static ip_address=192.168.1.100/24
static routers=192.168.1.1
static domain_name_servers=8.8.8.8 8.8.4.4
```

Datei speichern (`Ctrl+O`, `Enter`, `Ctrl+X`) und neu starten:

```bash
sudo reboot
```

---

## Schritt 3: Konfiguration ueberpruefen

Nach dem Neustart:

```bash
# IP-Adresse pruefen
ip addr show

# Gateway erreichbar?
ping -c 3 192.168.1.1

# Internet erreichbar?
ping -c 3 google.com
```

---

## Wichtige Hinweise

- **IP-Adresse ausserhalb des DHCP-Bereichs waehlen** — Pruefe in deinem Router, welcher Bereich fuer DHCP reserviert ist (z.B. `.100-.200`), und waehle eine Adresse ausserhalb davon (z.B. `.50`).
- **IP-Konflikt vermeiden** — Stelle sicher, dass kein anderes Geraet im Netzwerk dieselbe IP nutzt.
- **Router-Variante** — Alternativ kannst du im Router eine feste IP per DHCP-Reservierung (MAC-Adresse) vergeben. Das ist oft die sauberste Loesung.
- **SSH-Zugang** — Wenn du per SSH verbunden bist, aendert sich nach dem Neustart die IP. Verbinde dich dann mit der neuen statischen IP.
