# Getting started

## Netzwerk erstellen

Der Erste Schritt ist es ein Persistentes Netzwerk zu erstellen.
Dieses nennen wir, in diesem Fall, "wirdockenan.local"

```docker
docker network create -d bridge wirdockenan
```

## Die Container

Wir haben 5 Ordner mit jeweils 5 docker Compose Dateien.
Die Unterpunkte geben aus, welche Images in jedem Container laufen.

- OpenLDAP
- phpLDAPadmin
- Nextcloud
  - Nextcloud
  - Mariadb
- Moodle
  - Moodle
  - Mariadb
- Redis
- adminer

### Logins

|              |           Benutzername           | Passwort |        Link:Port       |
|:------------:|:--------------------------------:|:--------:|:----------------------:|
|   openLDAP   | cn=admin,dc=wirdockenan,dc=local |   admin  |          :389          |
| phpLDAPadmin |                /                 |          | <http://localhost:8080/> |
|   Nextcloud  |              Admin               |   Admin  |  <http://localhost:88/>  |
|  nc-mariadb  |              NcUser              |  NcUser  |          :3366         |
|    Moodle    |                                  |          | <https://localhost:443/> |
|  mo-mariadb  |              MoUser              |  MoUser  |          :3306         |
|     redis    |               redis              |     /    |                        |
|    adminer   |               root               |   Admin  | <http://localhost:8008/> |

## OpenLDAP und phpLDAPadmin

Beginnen wir mit dem OpenLDAP als Backend und dem phpLDAPadmin als Frontend.

Wir loggen uns als Admin im LDAP ein, dort erstellen wir eine neue Gruppe in unserer Domaine

1. "Create a Child entry"
2. "Organisational Unit" Diese Unit nennen wir "groups" und erstellen sie
3. In ou=groups erstellen wir verschiedene Posix Group: "administrator; developers; users"
4. diese "Posix Groups" können wir mit den jeweiligen "Generic: User Account" füllen
5. Dann fügen wir noch eine memberUid hinzu, die gleich dem UID der jeweiligen Nutzer ist.

Damit haben wir dann ein LDAP Server mit Gruppen und Usern.

## Nextcloud

Die erste Initialisierung von Nextcloud ist dafür da, um einen Admin Account zu erstellen und die Datenbank zu verbinden.
Unser Gewählter Admin Account ist in der Login Liste ist zu finden.
Verbindung zu Mariadb wird hergestellt indem man den Nutzer und das Passwort, sowie die entsprechende Datenbank und den mariadb-server:port einträgt.

Um die LDAP integration in Nextcloud hinzuzufügen. Muss man zuerst die LDAP-Integrations App aktivieren unter dem Reiter "Apps".
Danach geht man in die Verwaltungseinstellungen, unter dem Reiter "Verwaltung" findet man nun die LDAP-Integration.
In der LDAP-Integration, trägt man den LDAP-Server ein, in diesem fall ist es der Container name "openldap", danach kann man den Port 389 ermitteln.
Unter den Anmeldedaten trägt man den Account ein, mit dem sich Nextcloud in LDAP einloggen soll.
In den darauffolgenden Seiten der konfiguration, Kann man genau bestimmen, welche Benutzergruppen zugriff auf Nextcloud haben und kann dies nach belieben ausfüllen.

Sobald alle Notwendigen Felder ausgefüllt sind und die Konfigurationen Benutzer und Gruppen gefunden hat, ist Nextcloud mit openldap integriert.

# Notizen

###### <https://github.com/Danoletto/Docker-Aufgabe/blob/c6b141ef3767e6f19a2376884f85dff0572cf602/Readme.md?plain=1#L59C1-L59C1>

## Nächste aufgaben

    -   Redis, wie integriere ich es mit Moodle?
        -   Redis + php.dll, kann ich soetwas installieren. Wenn ja, wie ?
        -   Funktioniert es überhaupt mit Nextcloud? Wie teste ich sowas.
    
    Funktioniert schonmal:
    -   LDAP Funktioniert, User können erstellt werden und man kann sich bei Moodle & Nextcloud anmelden.
    -   Mariadb Speichert alle Datenbanken so wie sie soll.
    -   Alle Container und Images sind persistent.
    -   Init = true" ist nicht gut, lol
    -   Alle Container befinden sich im selben Netz.

#### Setup MariaDB

```Docker
docker run -d --name moodledb -v mariadb-data:/var/lib/mariadb --network back_Netzwerk -e "MARIADB_ALLOW_EMPTY_ROOT_PASSWORD=true" -e MARIADB_USER=Admin -e "MARIADB_PASSWORD=Admin" -e "MARIADB_DATABASE=moodle" mariadb
```

//not necessary volumes get created if they don't exsist
Setup MariaDB Volume:    docker volume create mariadb-data
Setup Moodle Volume:    docker volume create moodle-data

#### Setup Moodle

```Docker
docker run -d --name moodle -p 81:8080 -p 443:8443 -v moodle-data:/bitnami/moodle --network back_Netzwerk -e MOODLE_DATABASE_HOST=moodledb -e MOODLE_DATABASE_USER=Admin -e MOODLE_DATABASE_PASSWORD=Admin -e MOODLE_DATABASE_NAME=moodle bitnami/moodle
```

## Befehlerklärung

```Docker
init: true
```

Beispiel Nextcloud, diese Zeile starten den installations Prozess von Nextcloud, bei einem Neustart des Containers, neu. ⚠️ erstmal rauslassen.

# Links

### <u>Docker</u>

<https://docs.docker.com/engine/reference/commandline/network_create/>

### <u>openLDAP</u>

<https://hub.docker.com/r/osixia/openldap>
<https://www.techrepublic.com/article/how-to-populate-an-ldap-server-with-users-and-groups-via-phpldapadmin/>

---

### <u>phpLDAPadmin</u>

<https://hub.docker.com/r/osixia/openldap>

---

### <u>mariadb</u>

<https://hub.docker.com/_/mariadb>
---

### <u>Nextcloud</u>

<https://hub.docker.com/_/nextcloud> <p>
How to connect Nextcloud with openLDAP <p>
<https://i12bretro.github.io/tutorials/0278.html>

Admin Privileges
<https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/admin_delegation_configuration.html>

Mehr Memory
<https://help.nextcloud.com/t/nextcloud-aio-docker-change-php-memory-limit-values/143392/13>

---

### <u>Moodle</u>

###### <https://hub.docker.com/r/bitnami/moodle>

Github zu docker-compose
<https://github.com/ubc/moodle-docker>

Moodle openLDAP integration
<https://docs.moodle.org/403/en/LDAP_authentication#Enabling_LDAP_authentication>

Redis installation
<https://docs.moodle.org/402/en/Redis_cache_store>
---

### <u>Redis</u>

###### <https://hub.docker.com/r/bitnami/redis>

Docker-Compose file
<https://geshan.com.np/blog/2022/01/redis-docker/>

## Git help

<https://blog.mergify.com/what-is-the-difference-between-a-merge-commit-a-squash/>
<https://stackoverflow.com/questions/2427238/what-is-the-difference-between-merge-squash-and-rebase>
<https://github.blog/2016-04-01-squash-your-commits/>
