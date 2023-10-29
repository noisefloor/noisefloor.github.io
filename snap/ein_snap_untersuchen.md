---
titel: ein snap Paket untersuchen
ursprungsdatum: 2023-10-28
letzte_änderung: 2023-10-29
---



# ein snap Paket untersuchen

snap Pakete bestehen aus einer einzelnen Datei mit der Dateiendung **.snap** . Diese Datei ist technisch gesehen ein SquashFS-Dateisystem, in das alle Dateien, die das snap enthält, verpackt sind.

Was in dem snap Paket enthalten ist kann man unter- / durchsuchen. Je nach dem, ob das snap ins System eingehängt ist oder nicht unterscheidet sich der Weg.

# eingehängtes snap Paket untersuchen

Den Inhalt eines snap Pakets, welches ins System eingehängt ist, zu unter- / durchsuchen benötigt keine weiteren Hilfsmittel. Man navigiert einfach zum Einhängepunkt, also **/snap** . Dann geht man den Ordner des gewünschten snaps, z.B. **firefox** für das Firefox snap. In diesem Ordner befindet sich immer ein Verzeichnis **current** , welches immer ein Link auf die aktuelle, installiert, eingehängte Version des snaps ist. Wechselt man in dieses Verzeichnis sieht man alle Dateien und Verzeichnisse, die zum snap gehören. Durch diese Verzeichnisse kann man navigieren, sich Dateien anzeigen lassen usw. Befindet man sich z.B. im Verzeichnis **/snap/firefox/current** kann man sich im Terminal über 

```shell
less firefox.desktop
```

die im snap enthaltene [.desktop-Datei](https://wiki.ubuntuusers.de/.desktop-Dateien/ ) anzeigen lassen.

# nicht eingehängtes snap Paket untersuchen

Ein snap Paket muss aber nicht zwingend eingehängt sein, um es zu untersuchen. Man kann auch heruntergeladen bzw. nicht eingehängte Pakete untersuchen. Komfortable geht dies, indem man Programme aus den [squashfs-tools-ng ](https://github.com/AgentD/squashfs-tools-ng) nutzt.

## Vorbereitung: squashfs-tools-ng installieren

Das Paket "squashfs-tools-ng" ist in den Paketquellen einiger Linux-Distribution enthalten. Eine Übersicht findet man auf der [Projektseite](https://github.com/AgentD/squashfs-tools-ng#installing).

Unter Ubuntu / Debian / Raspberry Pi OS kann man das Paket über

```shell
sudo apt install squashfs-tools-ng
```

installieren.

## Inhalt anzeigen lassen

Nach der Installation von "squashfs-tools-ng" steht unter anderem das Programm `rdsquashfs` zur Verfügung. Mit dessen Hilfe kann man sich den Inhalt eines SquashFS-Dateisystems - und damit auch eines snaps - anzeigen lassen.

Zuerst wechselt man in der Verzeichnis, wo das snap Paket liegt. Unter Ubuntu werden regulär installierte snaps unter **/var/lib/snapd/snaps** gespeichert. Liegt dort z.B. das snap **core22_858.snap** kann man sich über

```shell
sudo rdsquashfs core22_858.snap -l /
```

das Wurzelverzeichnis diese snaps anzeigen lassen. Das `sudo` ist hier notwendig, weil die snaps in **/var/lib/snapdsnaps** zum Besitzer und zur Gruppe *root* gehören und die Dateirechte mit `300`  restriktiv gesetzt sind. Hätte man ein snap manuell heruntergeladen und z.B. im eigenen Home-Verzeichnis gespeichert, wären das Ausführen des Befehls mit `sudo` nicht notwendig.

Die Option `-l` des Programm `rdsquashfs` gibt an, welches Verzeichnis ausgegeben werden sollte. Möchte man sich z.B. alle ausführbaren Programme ausgeben lassen, die in diesem snap enthalten sind, ruft man den Befehl

```shell
sudo rdsquashfs core22_858.snap -l /usr/bin
```

auf.

Man kann sich mit Hilfe von `rdsquashfs` auch Dateien aus dem snap ausgeben lasse. Dazu muss man das Programm mit der Option `-c` gefolgt vom vollen Pfad und Dateinamen aufrufen. Im folgenden Beispiel würde die Datei **/etc/systemd/journald.conf** aus dem snap aufgelistet:

```shell
sudo rdsquashfs core22_858.snap -c /etc/systemd/journald.conf
```

Man kann sich auch einfach alle Dateien und Verzeichnisse aus dem snap auflisten lassen:

```shell
sudo rdsquashfs core22_858.snap -d
```

`rdsquashfs` kennt noch einige weitere Optionen, unter anderem zum Entpacken von einzelnen oder allen Dateien aus dem snap Paket. Diese kann man sich über den Aufruf von

```shell
rdsquashfs -h
#oder
man rdsquashfs
```

anzeigen lasssen.

## zwei snap Pakete vergleichen

Die "squashfs-tool-ng" beinhalten auch das Programm `sqfsdiff` mit dem sich der Inhalt von zwei SquashFS-Dateisystemen und damit auch von zwei snap Paketen vergleichen lässt. Im folgenden Beispiel würden zwei Version des core22 snaps, welche im aktuellen Verzeichnis gespeichert sind, vergleichen:

```shell
sudo sqfsdiff --old core22_858.snap --new core22_864.snap
```

Je nach dem, wie viele Dateien und Verzeichnisse im snap Paket sind bzw. sich geändert haben, kann die Ausgabe sehr lang sein. Es gibt verschiedene Arten von Unterschieden:

* Lautet eine Ausgabezeile `regular file ./pfad/zur datei/dateiname differs` bedeutet dies, dass die Datei und der Dateipfad in beiden snaps identisch ist, sich aber der Inhalt der Datei verändert hat.
* Lautet eine Ausgabezeile  `> /pfad/zur/datei/dateiname`bedeutet dies, diese diese Datei mit diesem Dateipfad in der neuen Datei enthalten ist, aber nicht in der alten.
* Lautet eine Ausgabezeile  `< /pfad/zur/datei/dateiname`bedeutet dies, diese diese Datei mit diesem Dateipfad in der alten Datei enthalten ist, aber nicht in der neue.
* Lautet eine Ausgabezeile `/pfad/zur/datei/dateiname has a different link target` bedeutet dies, dass Datei und Dateipfad in beiden snaps enthalten ist, aber sich die Verlinkung innerhalb des SquashFS-Dateisystem geändert hat.

`sqfsdiff` kennt eine Reihe von Optionen, um die Vergleich einzuschränken. Die Optionen kann man sich über den Aufruf von

```shell
sqfsdiff -h
#oder
man sqfsdiff
```

anzeigen lassen.

# Links

## snap

* [Startseite](https://snapcraft.io/docs) der offiziellen Dokumentation zu snaps

## squashfs

* [Wikipediaartikel](https://de.wikipedia.org/wiki/SquashFS) zu SquashFS
* [SquashFS Howto](https://tldp.org/HOWTO/SquashFS-HOWTO/whatis.html) beim The Linux Documentation Projekt
* [SquashFS Dokumentation](https://docs.kernel.org/filesystems/squashfs.html) des Linux Kernel

## squashfs-tools-ng

* [Projektseite](https://github.com/AgentD/squashfs-tools-ng) von squashfs-tools-ng
