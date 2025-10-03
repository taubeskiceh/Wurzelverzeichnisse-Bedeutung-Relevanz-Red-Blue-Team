# Wurzelverzeichnisse-Bedeutung-Relevanz-f-r-Red-Blue-Team
Ziel: Schnelle Referenz für System-Admins und Sicherheits-Teams — erklärt die wichtigsten Root-Verzeichnisse, markiert kritische Einträge und nennt spezielle Bereiche, die für Red- und Blue-Team-Aktivitäten besonders interessant sind.

Legende (Schnell)

🔴 Kritisch — Änderungen können Systemverfügbarkeit oder Sicherheit beeinträchtigen (Boot, Auth, Konfigurationen).

🟠 Wichtig — Betriebssystem-relevant; falsche Änderungen führen zu Problemen.

🟢 Normal / Sicher anschauen — meist nutzerbezogene oder temporäre Bereiche.

🛡️ Blue-Team relevant — gute Orte zum Monitoring, Forensik, Härtung.

🕵️ Red-Team relevant — gute Orte, um Persistenz, Spuren, oder Datensuche zu prüfen.

# Tabelle: Verzeichnis — Kurzbeschreibung — Markierung

## Verzeichnis	Kurzbeschreibung	Markierung / Hinweise

/	Root-Wurzel des Dateisystems	🟠 Systemkritisch (Basis)
. / ..	Aktuelles / übergeordnetes Verzeichnis (Navigation)	🟢
bin -> usr/bin	Essentielle User-Binaries (ls, bash)	🟠 (Symlink auf /usr/bin)
boot	Kernel, initramfs, Bootloader-Konfigs	🔴 (kritisch) — NICHT unbedacht ändern
cdrom	Mountpoint für CD/DVD	🟢 (selten genutzt)
dev	Gerätedateien (Kernel-managed)	🟠 — wichtig, aber dynamisch
etc	Systemweite Konfigurationen (/etc/passwd, sshd_config)	🔴 (kritisch) — zentral für Härtung & Forensik
home	Nutzerverzeichnisse (/home/<user>)	🟢 — Blue-Team: Nutzerdaten, Logs; Red-Team: Suche nach ssh-keys/passwords
lib, lib64	Shared-Libraries	🟠 — Abhängigkeiten; kompromittierte Libs = gefährlich
lost+found	fsck-Wiederherstellungsbereich	🟢 (nur nach fsck relevant)
media	Automount für Wechselmedien	🟢 — Blue-Team: USB-Forensik
mnt	Temporäre Mounts für Admins	🟢 — gut für manuelle Tests
opt	Optionale/Proprietäre Software	🟠 — 3rd-party-Apps, Aufmerksamkeit nötig
proc	Pseudo-FS: Prozess- und Kernel-Info	🟠 — nur lesbar, live-System-Info
root	Home-Verzeichnis von root	🔴 — sensible Daten, SSH-Keys, Scripts
run	Laufzeitdaten (PIDs, Sockets)	🟠 — Blue-Team: Sockets, PID-Files überwachen
sbin -> usr/sbin	System-Binaries (Admin-Tools)	🟠 — nur root-Tools
snap	Snap-Anwendungen & Mounts	🟠 — Snap-bezogene Angriffsfläche möglich
srv	Servicedaten (z. B. Web-/FTP-Daten)	🟠 — Blue/Red: Angriffs-/Daten-Location
swap.img	Swap-Datei (Auslagerung)	🟠 — enthält potentiell sensitive Klartext-Daten im RAM-Fallback
sys	Pseudo-FS: Kernel/Hardware-Info	🟠 — Live-Hardware-Ansicht
tmp	Temporäre Dateien (Sticky-Bit gesetzt)	🟢 — Blue-Team: Temp-Forensik; Red-Team: temporäre Persistenz versuchen
usr	Userland: /usr/bin, /usr/lib, /usr/share	🟠 — größter Programmsatz; Prüfpunkte für Integrität
var	Variable Daten: Logs, Caches, Mail, DBs	🔴 (kritisch) — wichtigste Quelle für Logs & Forensik
Welche Einträge sind kritisch (Kurz)

🔴 /boot: Kernel & Bootloader — Änderung kann System unbootbar machen.

🔴 /etc: zentrale Systemkonfiguration (Dienste, Auth, Netz).

🔴 /var: Logs und Laufzeitdaten — für Forensik unverzichtbar.

🔴 /root: root-homedir (SSH-Keys, sensible Scripts).

Warum kritisch? Die Integrität dieser Verzeichnisse bestimmt, ob das System startet, wie es sich authentifiziert und welche Spuren (Logs) vorhanden sind.

## Interessante Bereiche für Blue-Team (🛡️)

/var/log — Hauptquelle für System-/Anwendungslogs (syslog, auth.log, nginx, systemd). Unverzichtbar für Incident Response.

/etc — Härtung: SSH (/etc/ssh/), sudoers, PAM, services. Prüfe Berechtigungen, unerwartete Änderungen.

/run, /proc, /sys — Laufzeitchecks, aktive Sockets, Prozesse (z. B. offene Ports, DLL-Injektionen, ungewöhnliche PIDs).

/home — Persistence-Mechanismen/konfigurierte SSH-Keys, user-spezifische Crontabs und gespeicherte Anmeldeinformationen.

/tmp, /var/tmp — temporäre Dateien als Angriffs-/Persistenzvektor; prüfe setuid/unnötige ausführbare Dateien.

Swap (swap.img) — Analyse mit Tools kann sensiblen RAM-Inhalt offenbaren (Credentials, Keys).

## Interessante Bereiche für Red-Team (🕵️)

/etc/ssh & ~/.ssh — SSH-Konfiguration und private Keys (leichte Persistence/Seitwärtsbewegung).

/opt, /usr/local, /srv — 3rd-party Installationen: häufig ungetestete Dienste oder SUID/ungeschützte Binaries.

/tmp & mkdir-basierte Temporärpfade — einfache Stellen, um Backdoors temporär abzulegen.

Cron-Jobs (/etc/cron.*, user-crontabs) — wiederkehrende Persistenz.

Log-Directorys (/var/log) — Log-Wiping / Tampering für Eraser-Techniken (Achtung: Forensische Indikatoren bleiben).

## Schnelle Befehle für Audit & Forensik

Letzte Änderungen in /etc prüfen:

sudo find /etc -type f -mtime -7 -ls

Größte Dateien in /var (Logs, DBs):

sudo du -ah /var | sort -rh | head -n 20

Offene Sockets & Prozesse (laufzeit):

sudo ss -tulpen
ps aux --sort=-%mem | head -n 20

### Suche nach privaten SSH-Keys (wichtiger auf forensischen Checks):

sudo grep -R "BEGIN RSA PRIVATE KEY" /home /root /etc 2>/dev/null

### Swap (falls vorhanden) zur Extraktion (nur mit Zustimmung und Vorsicht):

sudo strings /swap.img | less

### Kurze Hardening- & Monitoring-Reminder

Sichere /etc/ssh/sshd_config: PasswordAuthentication no; PermitRootLogin no (oder stark eingeschränkt).

Logs zentralisieren (z. B. remote syslog/SIEM) — verhindert lokale Manipulation.

Setze Dateisystem-Integritätsprüfungen (AIDE, tripwire) auf /boot, /etc, /usr.

Beschränke Schreibrechte auf kritische Verzeichnisse; überwache chown/chmod-Änderungen.
