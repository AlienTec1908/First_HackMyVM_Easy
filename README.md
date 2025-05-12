# First - HackMyVM (Easy) - Writeup

![First Icon](First.png)

Dieses Repository enthält einen zusammengefassten Bericht über die Kompromittierung der HackMyVM-Maschine "First" (Schwierigkeitsgrad: Easy).

## Metadaten

*   **Maschine:** First (HackMyVM - Easy)
*   **Link zur VM:** [https://hackmyvm.eu/machines/machine.php?vm=First](https://hackmyvm.eu/machines/machine.php?vm=First)
*   **Autor des Writeups:** DarkSpirit
*   **Datum:** 2023-07-10
*   **Original Writeup:** [https://alientec1908.github.io/First_HackMyVM_Easy/](https://alientec1908.github.io/First_HackMyVM_Easy/)

## Zusammenfassung des Angriffspfads

Der Einstieg erfolgte durch Steganographie auf einer Bilddatei (`first_Logo.jpg`) mittels `stegseek` und der `rockyou.txt`-Wortliste. Dies enthüllte das Passwort `firstgurl1` und eine Base64-kodierte Nachricht. Die Dekodierung der Nachricht lieferte einen Hinweis und einen Hex-kodierten Pfad. Die Konvertierung des Hex-Strings ergab den Pfad `/t0d0_l1st_f0r_f1r5t`.

Die Enumeration dieses Verzeichnisses (vermutlich mit `gobuster`) führte zur Entdeckung einer Webshell (`ben.phtml` im Unterverzeichnis `/uploads/`) oder einer Möglichkeit, diese hochzuladen. Über den `cmd`-Parameter der Webshell wurde eine Reverse Shell als Benutzer `www-data` mittels `nc` aufgebaut und stabilisiert (`python pty`).

Die Analyse der `sudo`-Rechte für `www-data` (impliziert durch die nächsten Schritte) ergab die Berechtigung, `neofetch` als Benutzer `first` auszuführen. Dies wurde durch das Erstellen einer bösartigen Konfigurationsdatei (`mktemp`, `echo`) und deren Ausführung mittels `sudo -u first neofetch --config <datei>` ausgenutzt, was zu einer Shell als Benutzer `first` führte.

Eine erneute Prüfung der `sudo`-Rechte als `first` offenbarte die Möglichkeit, das benutzerdefinierte Programm `/usr/bin/secret` auszuführen. Nach Eingabe des hartkodierten Passworts `12345678910` konnte über das Programm der Befehl `bash` ausgeführt werden, was zu einer Root-Shell führte. Anschließend wurden die User- und Root-Flags ausgelesen.

## Verwendete Tools (Auswahl)

*   `stegseek`
*   `cat`
*   `echo`
*   `base64`
*   `gobuster`
*   `nc` (netcat)
*   `python3`
*   `pty` (Python module)
*   `export`
*   `ls`
*   `sudo`
*   `mktemp`
*   `neofetch`
*   `chmod`
*   `id`
*   `vi` (implied)
*   `secret` (custom binary)
*   `cd`

## Angriffsschritte (Übersicht)

1.  **Reconnaissance (Stego):** `stegseek` auf `first_Logo.jpg` mit `rockyou.txt` -> Passwort `firstgurl1` & Base64-Nachricht.
2.  **Decoding:** Base64-Nachricht decodieren (`base64 -d`), Hex-String extrahieren.
3.  **Decoding (Hex):** Hex-String (`2f...74`) zu ASCII konvertieren -> Pfad `/t0d0_l1st_f0r_f1r5t`.
4.  **Web Enumeration:** `gobuster` auf `/t0d0_l1st_f0r_f1r5t/` (impliziert) -> findet `/uploads/ben.phtml`.
5.  **Initial Access:** Reverse Shell via Webshell (`ben.phtml?cmd=...bash...`) und `nc`-Listener -> Shell als `www-data`.
6.  **Shell Stabilization:** `python3 -c 'import pty; ...'`, `export TERM=xterm`.
7.  **Privilege Escalation (www-data -> first):** `sudo -l` (impliziert) zeigt `(first) /bin/neofetch`. Erstellen einer bösartigen Config (`mktemp`, `echo 'exec /bin/sh' > $TF`), Berechtigungen setzen (`chmod 777 $TF`), Ausführen mit `sudo -u first neofetch --config $TF` -> Shell als `first`.
8.  **Privilege Escalation (first -> root):** `sudo -l` (impliziert) zeigt `sudo secret`. Ausführen mit `sudo secret`, Passwort `12345678910` eingeben, Befehl `bash` eingeben -> Shell als `root`.
9.  **Flags:** `cat /home/first/user.txt` und `cat /root/r00t.txt` auslesen.

## Flags

*   **User Flag (`/home/first/user.txt`):** `3120a57478d631a5ef82ef5d96146389`
*   **Root Flag (`/root/r00t.txt`):** `477d9a6aa33e3818ced1ad3015b53b43`

---

## Disclaimer

Die hier dargestellten Informationen und Techniken dienen ausschließlich Bildungs- und Forschungszwecken im Bereich der Cybersicherheit. Die beschriebenen Methoden dürfen nur auf Systemen angewendet werden, für die eine ausdrückliche Genehmigung vorliegt (z.B. in CTF-Umgebungen wie HackMyVM, Penetrationstests mit schriftlicher Erlaubnis). Das unbefugte Eindringen in fremde Computersysteme ist illegal und strafbar. Die Autoren übernehmen keine Haftung für missbräuchliche Verwendung der bereitgestellten Informationen. Handeln Sie stets legal und ethisch verantwortlich.

---
