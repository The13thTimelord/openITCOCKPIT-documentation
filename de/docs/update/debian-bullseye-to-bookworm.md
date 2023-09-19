# Update von Debian Bullseye (11) auf Debian Bookworm (12)

!!! danger "Backup"
    Bevor Sie beginnen, stellen Sie sicher ein **funktionierendes Backup** Ihres Systems gemacht zu haben!

Führen Sie alle Kommandos als `root` Benutzer aus.

Um immer die aktuelle Version von openITCOCKPIT verwenden zu können, ist es wichtig, dass zugrundeliegende Betriebssystem aktuell zu halten.
Mit dieser Anleitung können Sie Ihr Debian Bullseye System auf Debian Bookworm aktualisieren.

##  Voraussetzungen
 -  openITCOCKPIT in der Version 4.x
 -  Keine Pakete die `lxd` beinhalten.

## Entfernen aller `lxd` Pakete
Sollten auf Ihrem System `lxd` Pakete installiert sein, müssen diese zuerst entfernt werden. Prüfen können Sie dies mit dem Befehl
```
apt list --installed | grep lxd
```

Sollten Pakete installiert sein, müssen diese zuerst entfernt werden
```
apt -y remove lxd*
```

## Installation aller Updates
Bevor Sie mit dem Update von Debian beginnen, stellen Sie bitte sicher, dass alle verfügbaren Updates installiert wurden.

```
apt update
apt -y full-upgrade
```

## Stoppen Sie den PHP-FPM Dienst
```
systemctl stop php7.4-fpm.service
```

## openITCOCKPIT Pakete ermitteln
Nun werden zuerst alle installierten Pakete von openITCOCKPIT ermittelt und in der Variable `openitcockpit_upd` gespeichert.
```
openitcockpit_upd=$(apt-mark showmanual | grep openitcockpit | xargs echo)" "$(apt-mark showauto | grep openitcockpit | xargs echo)
```

## Paketquellen ändern
Nun können die Paketquellen auf den nächsten Debian-Release geändert werden
```
echo "deb http://deb.debian.org/debian bookworm main contrib non-free" > /etc/apt/sources.list
echo "deb-src http://deb.debian.org/debian bookworm main contrib non-free" >> /etc/apt/sources.list
echo "deb http://security.debian.org/debian-security bookworm-security main contrib non-free" >> /etc/apt/sources.list
echo "deb-src http://security.debian.org/debian-security bookworm-security main contrib non-free" >> /etc/apt/sources.list
echo "deb http://deb.debian.org/debian bookworm-updates main contrib non-free" >> /etc/apt/sources.list
echo "deb-src http://deb.debian.org/debian bookworm-updates main contrib non-free" >> /etc/apt/sources.list

sed -i 's/bullseye/bookworm/g' /etc/apt/sources.list.d/*.list
```

## Upgrade durchführen
Jetzt wird das eigentliche Upgrade von Debian 10 auf 11 durchgeführt.
```
apt update
apt -y full-upgrade $openitcockpit_upd php-json php-curl php-gearman php-xml
```

### Update von PHP
Nun werden alle PHP Pakete aktualisiert.

```
dpkg -l | awk '/php7.4/ { print $2}' | sed 's/php7.4/php8.2/' | xargs apt-get install -y
dpkg -l | awk '/php7.4/ { print $2}' | xargs apt-get purge -y
```

### Zeitzone in php.ini anpassen

```
date.timezone = Europe/Berlin
```

### PHP FPM (neu-)starten

```
systemctl restart php8.2-fpm.service
```

### Konfiguration aktualisieren
Im letzten Schritt werden alle Konfigurationsdateien aktualisiert und bei Bedarf neu generiert.

#### openITCOCKPIT Master
Wenn Sie das Update auf einem openITCOCKPIT Master System ausführen, nutzen Sie den folgenden Befehl
```
openitcockpit-update --cc
```

#### openITCOCKPIT Satellit
Auf einem openITCOCKPIT Satellit nutzen Sie den folgenden Befehl
```
/opt/openitc/frontend/UPDATE.sh
```

Um das Update abzuschließen, wird ein Neustart empfohlen
```
reboot
```
