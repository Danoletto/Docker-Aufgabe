# Notizen

###### https://github.com/Danoletto/Docker-Aufgabe/blob/c6b141ef3767e6f19a2376884f85dff0572cf602/Readme.md?plain=1#L59C1-L59C1

## Nächste aufgaben:
    -   Moodle intgrierung fertigstellen.
    -   Nextcloud Server Fehler fixen
    -   Redis, wie integriere ich es mit Moodle?
        -   Redis + php.dll, kann ich soetwas installieren. Wenn ja, wie ?
        -   Funktioniert es überhaupt mit Nextcloud? Wie teste ich sowas.
    
    Funktioniert schonmal:
    -   LDAP Funktioniert, User können erstellt werden und man kann sich bei Moodle & Nextcloud anmelden.
    -   Mariadb Speichert alle Datenbanken so wie sie soll.
    -   Alle Container und Images sind persistent.
    -   Init = true" ist nicht gut, lol
    -   Alle Container befinden sich im selben Netz.


#### Setup MariaDB:
```Docker
docker run -d --name moodledb -v mariadb-data:/var/lib/mariadb --network back_Netzwerk -e "MARIADB_ALLOW_EMPTY_ROOT_PASSWORD=true" -e MARIADB_USER=Admin -e "MARIADB_PASSWORD=Admin" -e "MARIADB_DATABASE=moodle" mariadb
```
//not necessary volumes get created if they don't exsist
Setup MariaDB Volume:    docker volume create mariadb-data
Setup Moodle Volume:    docker volume create moodle-data
#### Setup Moodle:
```Docker
docker run -d --name moodle -p 81:8080 -p 443:8443 -v moodle-data:/bitnami/moodle --network back_Netzwerk -e MOODLE_DATABASE_HOST=moodledb -e MOODLE_DATABASE_USER=Admin -e MOODLE_DATABASE_PASSWORD=Admin -e MOODLE_DATABASE_NAME=moodle bitnami/moodle
```
## Befehlerklärung:
```Docker
init: true
```
Beispiel Nextcloud, diese Zeile starten den installations Prozess von Nextcloud, bei einem Neustart des Containers, neu. ⚠️ erstmal rauslassen.

## OpenLDAP Login

login|password
-|-
cn=admin,dc=wirdockenan,dc=local|Admin


login|password
-|-
admin|Admin123!

# Links

### <u>openLDAP</u>
###### https://hub.docker.com/r/osixia/openldap
https://www.techrepublic.com/article/how-to-populate-an-ldap-server-with-users-and-groups-via-phpldapadmin/

---
### <u>phpLDAPadmin</u>
###### https://hub.docker.com/r/osixia/openldap

---
### <u>mariadb</u>
######  https://hub.docker.com/_/mariadb
---
### <u>Nextcloud</u>
###### https://hub.docker.com/_/nextcloud
How to connect Nextcloud with openLDAP
https://i12bretro.github.io/tutorials/0278.html


Admin Privileges
https://docs.nextcloud.com/server/latest/admin_manual/configuration_server/admin_delegation_configuration.html


Mehr Memory
https://help.nextcloud.com/t/nextcloud-aio-docker-change-php-memory-limit-values/143392/13

---
### <u>Moodle</u>
###### https://hub.docker.com/r/bitnami/moodle

Github zu docker-compose
https://github.com/ubc/moodle-docker

Moodle openLDAP integration
https://docs.moodle.org/403/en/LDAP_authentication#Enabling_LDAP_authentication

Redis installation
https://docs.moodle.org/402/en/Redis_cache_store
---
### <u>Redis</u>
###### https://hub.docker.com/_/redis

Docker-Compose file
https://geshan.com.np/blog/2022/01/redis-docker/


## Git help

https://blog.mergify.com/what-is-the-difference-between-a-merge-commit-a-squash/
https://stackoverflow.com/questions/2427238/what-is-the-difference-between-merge-squash-and-rebase
https://github.blog/2016-04-01-squash-your-commits/