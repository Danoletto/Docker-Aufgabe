# Notizen


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
Beispiel Nexcloud, diese Zeile starten den installations Prozess von Nextcloud, bei einem Neustart des Containers, neu. \ erstmal rauslassen.

## OpenLDAP

|login:|cn=admin,cd=example,cd=org|
| - | - 
|password:|Admin|








# Links

### <u>openLDAP</u>
###### https://hub.docker.com/r/osixia/openldap
https://www.techrepublic.com/article/how-to-populate-an-ldap-server-with-users-and-groups-via-phpldapadmin/

---
### <u>phpLDAPadmin</u>
###### https://hub.docker.com/r/osixia/openldap

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

Moodle openLDAP integration
https://docs.moodle.org/403/en/LDAP_authentication#Enabling_LDAP_authentication

---
### <u>Redis</u>
###### https://hub.docker.com/_/redis

Docker-Compose file
https://geshan.com.np/blog/2022/01/redis-docker/