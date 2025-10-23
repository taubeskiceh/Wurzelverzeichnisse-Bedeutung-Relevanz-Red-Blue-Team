# 🧭 Wurzelverzeichnisse – Bedeutung & Relevanz für Red- und Blue-Team

**Ziel:**  
Schnelle Referenz für **System-Administratoren**, **Blue-Teams (Defense)** und **Red-Teams (Offense)**.  
Dieses Dokument erklärt die wichtigsten Root-Verzeichnisse eines Linux-Systems, markiert **kritische Punkte** und zeigt, welche Bereiche für **Forensik, Monitoring oder Angriffe** besonders relevant sind.

---

## 📚 Inhaltsverzeichnis

1. [Legende](#-legende)
2. [Hauptverzeichnisse – Übersicht & Bewertung](#-hauptverzeichnisse--übersicht--bewertung)
3. [Kritische Verzeichnisse – Kurzüberblick](#️-kritische-verzeichnisse--kurzüberblick)
4. [Interessante Bereiche für Blue-Team](#️-interessante-bereiche-für-blue-team)
5. [Interessante Bereiche für Red-Team](#-interessante-bereiche-für-red-team)
6. [Schnelle Audit- & Forensik-Befehle](#-schnelle-audit--forensik-befehle)
7. [Hardening & Monitoring Reminder](#-hardening--monitoring-reminder)
8. [Tools & Referenzen](#-weiterführende-tools--referenzen)
9. [Kurzfazit](#-kurzfazit)

---

## 🔖 Legende

| Symbol | Bedeutung |
|:------:|------------|
| 🔴 | **Kritisch** – Änderungen können Systemverfügbarkeit oder Sicherheit beeinträchtigen (Boot, Auth, Konfigurationen). |
| 🟠 | **Wichtig** – Betriebssystemrelevant; falsche Änderungen führen zu Fehlverhalten. |
| 🟢 | **Normal / Sicher** – meist nutzerbezogene oder temporäre Bereiche. |
| 🛡️ | **Blue-Team-Relevanz** – Orte für Monitoring, Forensik, Härtung. |
| 🕵️ | **Red-Team-Relevanz** – Orte für Persistenz, Spuren oder Datensuche. |

---

## 📁 Hauptverzeichnisse – Übersicht & Bewertung

| Verzeichnis | Kurzbeschreibung | Bewertung / Hinweise |
|:-------------|:----------------|:---------------------|
| `/` | Root-Wurzel des Dateisystems | 🟠 Systemkritisch (Basis) |
| `.` / `..` | Aktuelles / übergeordnetes Verzeichnis | 🟢 Navigation |
| `/bin → /usr/bin` | Essentielle User-Binaries (`ls`, `bash`) | 🟠 Symlink auf `/usr/bin` |
| `/boot` | Kernel, Initramfs, Bootloader-Konfigs | 🔴 **Kritisch – NICHT unbedacht ändern** |
| `/cdrom` | Mountpoint für CD/DVD | 🟢 Selten genutzt |
| `/dev` | Gerätedateien (Kernel-managed) | 🟠 Wichtig, dynamisch |
| `/etc` | Systemweite Konfigurationen (`passwd`, `sshd_config`) | 🔴 **Zentral für Härtung & Forensik** 🛡️ |
| `/home` | Nutzerverzeichnisse (`/home/<user>`) | 🟢 **🛡️ Blue:** Nutzerdaten & Logs — **🕵️ Red:** SSH-Keys/Passwords |
| `/lib`, `/lib64` | Shared Libraries | 🟠 Kompromittierte Libs = gefährlich |
| `/lost+found` | Wiederherstellungsbereich nach fsck | 🟢 Nur bei Fehlern relevant |
| `/media` | Automount für Wechselmedien | 🟢 🛡️ USB-Forensik |
| `/mnt` | Temporäre Mounts (manuell) | 🟢 Gut für Tests |
| `/opt` | Optionale / Proprietäre Software | 🟠 🕵️ Häufige 3rd-party-Angriffsfläche |
| `/proc` | Pseudo-FS: Prozess-/Kernel-Info | 🟠 Laufzeitinformationen |
| `/root` | Home-Verzeichnis von Root | 🔴 **Sensible Daten, SSH-Keys, Scripts** |
| `/run` | Laufzeitdaten (PIDs, Sockets) | 🟠 🛡️ Überwache PID-Files & Sockets |
| `/sbin → /usr/sbin` | System-Binaries (Admin-Tools) | 🟠 Root-only Tools |
| `/snap` | Snap-Anwendungen & Mounts | 🟠 Angriffsfläche möglich |
| `/srv` | Servicedaten (z. B. Web-/FTP-Daten) | 🟠 🛡️🕵️ Datenquelle & Angriffsfläche |
| `/swap.img` | Swap-Datei | 🟠 🛡️ Sensible RAM-Reste möglich |
| `/sys` | Kernel/Hardware-Info | 🟠 Laufzeit-Hardware-Daten |
| `/tmp` | Temporäre Dateien | 🟢 🛡️🕵️ **Forensik / Temporäre Persistenz** |
| `/usr` | Userland: Programme & Libraries | 🟠 Prüfpunkte für Integrität |
| `/var` | Logs, Caches, DBs, Mails | 🔴 **Kritisch – Forensik-Hauptquelle** 🛡️ |

---

## ⚠️ Kritische Verzeichnisse – Kurzüberblick

| Verzeichnis | Grund der Kritikalität |
|:-------------|:----------------------|
| `/boot` | Kernel & Bootloader – Änderung = System unbootbar |
| `/etc` | Systemkonfigurationen (Auth, Dienste, Netz) |
| `/var` | Logs & Laufzeitdaten – Forensik & Spuren |
| `/root` | Root-Homedir – SSH-Keys, Scripte, Credentials |

> 🧠 **Hinweis:**  
> Die Integrität dieser Verzeichnisse bestimmt, ob das System startet, sich authentifiziert und Spuren nachvollziehbar bleiben.

---

## 🛡️ Interessante Bereiche für Blue-Team

| Bereich | Fokus / Empfehlung |
|:---------|:-------------------|
| `/var/log` | **Zentrale Logquelle:** `syslog`, `auth.log`, `nginx`, `systemd`. Unverzichtbar für Incident Response. |
| `/etc` | **Härtung:** SSH (`/etc/ssh/`), `sudoers`, PAM, Dienste-Konfigs. Berechtigungen & Änderungen prüfen. |
| `/run`, `/proc`, `/sys` | Laufzeitchecks, Prozesse, Ports, DLL-Injektionen, ungewöhnliche PIDs. |
| `/home` | Nutzerbezogene Persistenz: `.ssh`, Crontabs, gespeicherte Credentials. |
| `/tmp`, `/var/tmp` | Temporäre Dateien – prüfe SetUID & ungewöhnliche Binaries. |
| `/swap.img` | Mögliche RAM-Spuren (Credentials, Keys) → **nur mit Zustimmung analysieren.** |

---

## 🕵️ Interessante Bereiche für Red-Team

| Bereich | Zweck / Nutzen |
|:---------|:---------------|
| `/etc/ssh`, `~/.ssh` | SSH-Konfiguration, Keys – **Persistence / Lateral Movement** |
| `/opt`, `/usr/local`, `/srv` | 3rd-party-Software, oft schwach gesichert oder SUID-Binaries |
| `/tmp` & ähnliche Pfade | Temporäre **Backdoors / Loader / Payloads** |
| `Cron-Jobs` (`/etc/cron.*`, User-Crontabs) | **Wiederkehrende Persistenz** |
| `/var/log` | **Log Manipulation / Erasure** – Spurenverwischung |

---

## 🧩 Schnelle Audit- & Forensik-Befehle

### 🔍 Letzte Änderungen in `/etc`
```bash
sudo find /etc -type f -mtime -7 -ls

## 📦 Größte Dateien in /var
sudo du -ah /var | sort -rh | head -n 20

## ⚙️ Laufzeitüberwachung (Prozesse & Sockets)
sudo ss -tulpen
ps aux --sort=-%mem | head -n 20

## 🔑 Suche nach privaten SSH-Keys
sudo grep -R "BEGIN RSA PRIVATE KEY" /home /root /etc 2>/dev/null

## 🧠 Swap-Datei prüfen (mit Vorsicht!)
sudo strings /swap.img | less

## 🧱 Hardening & Monitoring Reminder
SSH härten:
PasswordAuthentication no
PermitRootLogin no (oder stark einschränken)

# Zentrale Logs:
Remote-Syslog / SIEM zur Vermeidung lokaler Manipulation.

# Integritätsprüfung:
Setze AIDE oder Tripwire für /boot, /etc, /usr.

# Rechtemanagement:
Schreibrechte auf kritische Verzeichnisse beschränken.
Überwache chown / chmod-Änderungen in Echtzeit.


## 🔗 Weiterführende Tools & Referenzen
Tool	            Beschreibung / Zweck
Lynis             Audit & Hardening Tool
Chkrootkit        Rootkit-Erkennung
AIDE              Integritätsprüfer
Sysdig / Falco    Laufzeitüberwachung
The Sleuth Kit    Forensische Analyse


## 🧠 Kurzfazit

Für Blue-Teams: Fokus auf /etc, /var, /home, /tmp, /run – hier entstehen Spuren, hier muss Härte hin.

Für Red-Teams: Fokus auf /etc/ssh, /opt, /srv, /tmp, /root – hier liegen Wege zu Persistenz und Daten.

Für Admins: Änderungen an /boot, /etc, /var nur mit Backup und Doku – jede Zeile zählt!



Autor: golubsource
Lizenz: MIT
