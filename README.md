# Einstieg

## Wissenstand

### Docker

Noch keine Erfahrung.

### Microservices

Noch keine Erfahrung.

### Dockerbefehle

| Command      | Definition                                        |
| ------------ | ------------------------------------------------- |
| docker start | startet einen oder mehrere nicht aktive Container |
| docker stop  | stoppt einen oder mehrere aktive Container        |
| docker build | Erstellt einen Image aus einem Dockerfile         |
| docker pull  | Ladet ein Image aus der Registry (z. B Dockerhub) |
| docker push  | Ladet ein Image in die Registry hoch              |
| docker run   | Befehl ausführen in einem Container               |
| docker rm    | Beendet und löscht Container                      |
| docker rmi   | löscht Image                                      |
| docker ps    | Zeigt alle laufende Container an                  |



# Docker Umgebung in der Cloud 

## Netzwerkplan

```
+-----------------------------------------------------------+
! Container wordpress Frontend Webserver - 18.217.240.32:80 !
! Container mariadb Datenbank - Keine Public-IP             !
! Container cAdvisor Monitoring - 18.217.240.32:8080        !
! Container Grafana Monitoring - 18.217.240.32:3000         !
+-----------------------------------------------------------+
! Docker                                                    !
+-----------------------------------------------------------+
! Cloud - AWS EC2 Amazon Linux 2                            !
+-----------------------------------------------------------+
! Notebook - Schulnetz 10.x.x.x                             !
+-----------------------------------------------------------+
```



## AWS Dienst Aufbau

Es wurde eine Virtuelle Cloud Umgebung über Amazons Cloud Hosting AWS erschaffen. Docker wurde auf einer **Amazon Linux 2** Virtuellen-Machine erstellt

Vorgehensweise:

1. Einen AWS-Account erstellen
2. AWS EC2 Instance erstellen mit einer Instanz nach Wahl
3. Den SSH-Key unbedingt herunterladen
4. Mittels PuTTYgen oder sonstige Key-Gen-Programm den SSH-Key vom vorherigen Schritt auf das Key-Gen Tool aufladen und es anschliessend als ein privates Key abspeichern
5. Im SSH Remote Client die Instanz Konfiguration vornehmen (zur Hilfe gibt es diesen Youtube Video: <https://www.youtube.com/watch?v=bi7ow5NGC-U>)
6. Mit einer SSH Remote Client  eine Verbindung zur Instanz aufbauen.



## Installation von Docker

Es wurde Docker auf einer Linux Virtuellen-Maschine installiert

Vorgehensweise:

1. Ausführen des Befehls zum Docker installieren

   ```
   sudo amazon-linux-extras install docker
   ```

   

2. Ausführen des Befehls zum Docker starten

   ```
   sudo service docker start
   ```



3. Optional kann der Standarduser 'ec2-user' in die Docker-Gruppe hinzugefügt werden. Dies erlaubt den User Docker-Befehle auszuführen ohne Adminrechte. Befehl:

   ```
   sudo usermod -a -G docker ec2-user
   ```



4. Reboot ausführen, damit es wirkt



5. sich als root anmelden und in Verzeichnis /var/lib/docker wechseln

   ```
   sudo su
   cd /var/lib/docker
   ```



6.  Ausführen des Befehls um die Berechtigung vom Ordner 'Volumes' zu ändern

   ```
   sudo chmod -R 755 volumes/
   ```



## Installation von Docker Compose

Mit Docker Compose lassen sich ganz einfach mehrere Container aufbauen.

Vorgehensweise:

1. Sich als Root anmelden



2. Docker-Compose Paket herunterladen. Es wurde mit folgendem Befehl das Paket bezogen

   ```
   curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
   ```



3. Befehl ausführen um die Berechtigung zu ändern

   ```
   chmod +x /usr/local/bin/docker-compose
   ```



4. Sich von Root abmelden



5. Testen ob docker-compose richtig installiert worden ist

   ```
   docker-compose -v
   ```

   



## Wordpress, MariaDB, Cadvisor, Grafana Installation

1. neues Verzeichnis erstellen und wechseln

   

2. mit einem Texteditor nach Wahl eine Dockerfile Datei erstellen

```
sudo <Texteditor> Dockerfile
```



3. Folgendes in die Datei ergänzen

```
FROM orchardup/php5
ADD . /code
```



4. File Abspeichern und Texteditor verlassen



5. Wieder ein neues File erstellen mit einem beliebigem Texteditor. Das neue File ist ein .yml und heisst: 'docker-compose.yml'



6. Es werden Docker-Compose Befehle eingetragen die dann ausgeführt werden können. Wordpress, MariaDB, Cadvisor und Grafana werden hier definiert. Im File sind diese Zeilen eingegeben worden. 

   ```
   version: '3'
   
   services:
    wordpress:
     image: wordpress
     links:
      - mariadb:mysql
     environment:
      - WORDPRESS_DB_PASSWORD=example
     ports:
      - 80:80
     volumes:
      - ./code:/code
      - ./html:/var/www/html
   
    mariadb:
     image: mariadb
     environment:
      - MYSQL_ROOT_PASSWORD=example
      - MYSQL_DATABASE=wordpress
     volumes:
      - ./database:/var/lib/mysql
   
    cadvisor:
     image: google/cadvisor
     ports:
      - 8080:8080
     restart: always
     volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
   
    grafana:
     image: grafana/grafana
     ports:
      - 3000:3000
     restart: always
     volumes:
      - /:/rootfs:ro
      - /var/run:/var/run:rw
      - /sys:/sys:ro
      - /var/lib/docker/:/var/lib/docker:ro
   
   ```

   

7. Die Datei abspeichern 



8. Befehl ausführen um die erstellte .yml Datei ausführen. Der Befehl erkennt das es eine .yml Datei und kann somit problemlos ausgeführt werden

   ```
   docker-compose up -d
   ```

Via den Webbrowser lässt sich diese Instanz als Wordpress Webseite anzeigen.

<http://18.217.240.32:80>



## Sicherheitsaspekte

Container laufen virtuell auf der AWS und sind somit isoliert.



### Monitoring

Für den Monitoring ist ein cAdvisor aufgerichtet worden und Grafana für Aktive Benachrichtigung aktiviert.



Dank cAdvisor lassen sich die einzelne Containers bewachen. Ressourcen die von den Container benutzt werden sind auf der Seite dann ersichtlich

cAdvisor: [http://18.217.240.32:8080](http://18.217.240.32:8080/)

Grafana: [http://18.217.240.32:3000](http://18.217.240.32:3000/)



### Aspekte der Containersicherung

Es gibt Möglichkeiten über den AWS Snapshots und Backups zu machen. 

#### Docker Images Backup

In Docker selbst können sich die Images einzeln backupen. Dazu kann der 'docker save'-Befehl verwendet werden. Dies erzeugt ein .tar zip Verzeichnis. Mit dem 'docker load'-Befehl kann sich das Backup wieder einspielen.



#### Docker Container Backup

Es gibt verschieden Methoden Docker Container abzusichern

- Einen neuen Docker Image auf einen Docker Container mit dem 'docker-commit'-Befehl ausführen
- Einen Export vom Docker Container Filesystem als .tar Zipordner mit dem 'docker export'-Befehl. Anschliessend lässt sich dies mit dem 'docker-import'-Befehl wiederherstellen.





#### Einschränken der Ressourcen

##### Memory begrenzen (oder erweitern)

Arbeitsspeicher der Container lassen sich begrenzen respektiv erweitern. Jedoch möchten wir in Bezug auf Sicherheit eher den Memory begrenzen.

```
docker run -it --memory="[beliebige Zahl][mb, g , etc]" [Container]
```

##### CPU begrenzen

Das gleich lässt sich auch für den CPU einstellen.

```
docker run --cpus"[beliebige Zahl]" [Container]
```



# Testfälle

| Testfall                                                     | Ergebnis                 |
| ------------------------------------------------------------ | ------------------------ |
| Verbindung auf die Webseite: <http://18.217.240.32:80>       | Die Seite wird angezeigt |
| Verbindung auf cAdvisor Monitoring Seite: <http://18.217.240.32:8080> | Die Seite wird angezeigt |
| Verbinudng auf Grafana Monitoring Seite: <http://18.217.240.32:3000> | Die Seite wird angezeigt |



# Bonus

## Vergleich Vorher Nachher

### Docker / Container

In diesem Modul konnte ich die Grundlage von Container und Docker erlernen. Ich verstehe nun was es genau ist und machen kann. Wie der Bestpractice umgesetzt werden kann und habe auch erkannt was für ein Potenzial Container besitzt. Es ist im Gegensatz zu Virtuellen Maschinen eine kompaktere Methode. 



### Microservice

Persönlich konnte ich noch nie mit Microservices arbeiten, deshalb habe ich dank diesem Modul einen enormen Zuwachs bekommen. Das Konzept verstehe ich nun und bin auch bewusst wie man dies umsetzen kann.



## Reflexion

Am Anfang war ich relativ unsicher wie ich an das ganz angehen wollte. Ich hatte bei Vagrant recht schnell verstanden wie ich mit den Befehlen umgehen kann. Bei Docker sind die Befehle von ihrer Funktion her sehr ähnlich, jedoch sind sie komplizierter auszuführen. Aus diesem Grund bin konnte keinen konkreten Fortschritt machen. Dann wurde mehr Zeit mit Docker investiert und langsam konnte ich es mehr verstehen.