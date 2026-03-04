# IP-Adresse auf LCD Display anzeigen (Raspberry Pi 4)

## Hardware-Voraussetzungen

- Raspberry Pi 4
- I2C LCD Display (z.B. 16x2 HD44780 mit I2C-Adapter)
- Verkabelung: **SDA** → GPIO 2, **SCL** → GPIO 3, **VCC** → 5V, **GND** → GND

---

## 1. I2C aktivieren

```bash
sudo raspi-config
```

Navigiere zu: **Interface Options → I2C → Enable**

---

## 2. Bibliothek installieren

```bash
sudo apt update
sudo apt install python3-smbus i2c-tools
pip3 install RPLCD
```

---

## 3. I2C-Adresse pruefen

```bash
i2cdetect -y 1
```

Typische Adresse: `0x27` oder `0x3F`.

---

## 4. Python-Script erstellen

Datei anlegen:

```bash
nano /home/pi/show_ip.py
```

Inhalt:

```python
#!/usr/bin/env python3
import socket
import time
from RPLCD.i2c import CharLCD

# I2C-Adresse anpassen falls noetig (0x27 oder 0x3F)
lcd = CharLCD(i2c_expander='PCF8574', address=0x27,
              port=1, cols=16, rows=2)

def get_ip():
    try:
        s = socket.socket(socket.AF_INET, socket.SOCK_DGRAM)
        s.connect(("8.8.8.8", 80))
        ip = s.getsockname()[0]
        s.close()
        return ip
    except Exception:
        return "Keine Verbindung"

lcd.clear()
lcd.write_string("IP-Adresse:")
lcd.cursor_pos = (1, 0)
lcd.write_string(get_ip())
```

Ausführen:

```bash
python3 /home/pi/show_ip.py
```

---

## 5. Automatisch beim Booten starten

### Variante 1: rc.local

In `/etc/rc.local` vor `exit 0` einfuegen:

```bash
sleep 10
python3 /home/pi/show_ip.py &
```

### Variante 2: systemd-Service (empfohlen)

Datei anlegen:

```bash
sudo nano /etc/systemd/system/show-ip.service
```

Inhalt:

```ini
[Unit]
Description=Show IP on LCD
After=network-online.target
Wants=network-online.target

[Service]
ExecStartPre=/bin/sleep 10
ExecStart=/usr/bin/python3 /home/pi/show_ip.py
Type=oneshot
User=root

[Install]
WantedBy=multi-user.target
```

Aktivieren:

```bash
sudo systemctl enable show-ip.service
```

> **Hinweis:** Das `sleep 10` gibt dem Netzwerk Zeit, sich zu verbinden, bevor die IP abgefragt wird.
