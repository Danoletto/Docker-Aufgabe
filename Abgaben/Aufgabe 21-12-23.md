# Abgabe 21.12.23
###### Jannik,Nomo,Dan


### MariaDB
```docker
version: '3'
networks:
  WirDockenAn:
    external: true
services: 
  db:
    image: mariadb:latest
    restart: always
    volumes:
      - ./data/mariadb-data:/var/lib/mariadb
      - ./data/mysql-data:/var/lib/mysql
    environment:
      MARIADB_ROOT_PASSWORD: Admin
      MARIADB_DATABASE: Nextcloud
      MARIADB_USER: NcUser
      MARIADB_PASSWORD: NcUser
    networks:
      - WirDockenAn
    ports:
      - 3306:3306

  adminer:
    image: adminer
    restart: always
    networks:
      - WirDockenAn
    ports:
      - 80:8080
```
### Moodle

```Docker
version: '3'
networks:
  WirDockenAn:
    external: true
services:
  moodle:
    image: bitnami/moodle:latest
    ports:
      - '82:8080'
      - '443:8443'
    environment:
      - MOODLE_DATABASE_HOST=mariadb
      - MOODLE_DATABASE_PORT=NUMBER 3306
      - MOODLE_DATABASE_NAME=Moodle
      - MOODLE_DATABASE_USER=MoUser
      - MOODLE_DATABASE_PASSWORD=MoUser
    volumes:
      - ./data/moodledata-data:/bitnami/moodledata
      - ./data/bitnami-data:/bitnami/apache/conf
      - ./data/moodle_data:/bitnami/moodle
    networks:
      - WirDockenAn
```
### Nextcloud
```Docker
version: '3'
networks:
  WirDockenAn:
    external: true
services:
  app:
    image: nextcloud:latest
  # Was machat das überhaupt? 
  # init: true
    restart: always
    container_name: Nextcloud
    volumes:
      - ./data/nextcloud:/var/www/html
    environment:
      - MYSQL_DATABASE Nextcloud
      - MYSQL_USER NcUser
      - MYSQL_PASSWORD NcUser
      - MYSQL_HOST mariadb
    networks:
      - WirDockenAn
    ports:
      - "81:80"
```

### openLDAP
```Docker
version: '3'
networks:
  Netzwerk:
    name: WirDockenAn
    driver: bridge
services:
  openldap:
    # latest, da es seit 3 Jahren keine updates bekommt.
    image: osixia/openldap:latest
    container_name: openldap2
    env_file: .env
    tty: true
    stdin_open: true
    volumes:
      - ./data/certificates:/var/lib/ldap
      - ./data/slapd/database:/etc/ldap/slapd.d
      - ./data/slapd/config:/container/service/slapd/assets/certs/ 
    ports:
      - "389:389"
      - "636:636"
    # For replication to work correctly, domainname and hostname must be
    # set correctly so that "hostname"."domainname" equates to the
    # fully-qualified domain name for the host.
    domainname: "wirdockenan.local"
    hostname: "ldap-server"
    networks:
      - Netzwerk
```

### phpLDAPadmin
```Docker
version: '3'
networks:
  WirDockenAn:
    external: true
services:
  phpldapadmin:
    # latest, da es seit 3 Jahren keine updates bekommt.
    image: osixia/phpldapadmin:latest
    container_name: phpldapadmin2
    environment:
      PHPLDAPADMIN_LDAP_HOSTS: "openldap"
      PHPLDAPADMIN_HTTPS: "false"
    volumes:
      - .data/phpldapadmin-data:/var/www/phpldapadmin
    ports:
      - "8080:80"
    networks:
      - WirDockenAn
```