# Abgabe 17.01.2024
## Jannik Grendel, Nomo Gillet, Dan Schlüsselburg WIT3D

## openLDAP

```docker
version: '3'
networks:
  wirdockenan.local:
    external: true
services:
  openldap:
    # latest, da es seit 3 Jahren keine updates bekommt.
    image: osixia/openldap:latest
    container_name: openldap
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
    hostname: "ldap.wirdockenan.de"
    networks:
      - wirdockenan.local
```

## openLDAP .env

```docker
LDAP_LOG_LEVEL: "256"
LDAP_ORGANISATION: "WirDockenAn GmbH"
# muss das klein?
LDAP_DOMAIN: "wirdockenan.local"
LDAP_BASE_DN: ""
LDAP_ADMIN_PASSWORD: "admin"
LDAP_CONFIG_PASSWORD: "config"
LDAP_READONLY_USER: "false"
#LDAP_READONLY_USER_USERNAME: "readonly"
#LDAP_READONLY_USER_PASSWORD: "readonly"
LDAP_RFC2307BIS_SCHEMA: "false"
LDAP_BACKEND: "mdb"
LDAP_TLS: "true"
LDAP_TLS_CRT_FILENAME: "ldap.crt"
LDAP_TLS_KEY_FILENAME: "ldap.key"
LDAP_TLS_DH_PARAM_FILENAME: "dhparam.pem"
LDAP_TLS_CA_CRT_FILENAME: "ca.crt"
LDAP_TLS_ENFORCE: "false"
LDAP_TLS_CIPHER_SUITE: "SECURE256:-VERS-SSL3.0"
LDAP_TLS_VERIFY_CLIENT: "demand"
LDAP_REPLICATION: "false"
#LDAP_REPLICATION_CONFIG_SYNCPROV: 'binddn="cn=admin,cn=config" bindmethod=simple credentials="$$LDAP_CONFIG_PASSWORD" searchbase="cn=config" type=refreshAndPersist retry="60 +" timeout=1 starttls=critical'
#LDAP_REPLICATION_DB_SYNCPROV: 'binddn="cn=admin,$$LDAP_BASE_DN" bindmethod=simple credentials="$$LDAP_ADMIN_PASSWORD" searchbase="$$LDAP_BASE_DN" type=refreshAndPersist interval=00:00:00:10 retry="60 +" timeout=1 starttls=critical'
#LDAP_REPLICATION_HOSTS: "#PYTHON2BASH:['ldap://ldap.example.org','ldap://ldap2.example.org']"
KEEP_EXISTING_CONFIG: "false"
LDAP_REMOVE_CONFIG_AFTER_SETUP: "true"
LDAP_SSL_HELPER_PREFIX: "ldap"
```

## phpLDAPadmin

```docker
version: '3'
networks:
  wirdockenan.local:
    external: true
services:
  phpldapadmin:
    # latest, da es seit 3 Jahren keine updates bekommt.
    image: osixia/phpldapadmin:latest
    container_name: phpldapadmin
    environment:
      PHPLDAPADMIN_LDAP_HOSTS: "openldap"
      PHPLDAPADMIN_HTTPS: "false"
    volumes:
      - ./data/phpldapadmin-data:/var/www/phpldapadmin
    ports:
      - "8080:80"
    networks:
      - wirdockenan.local
```

## Nextcloud

```docker
version: '3'
networks:
  wirdockenan.local:
    external: true
services:
  app:
    image: nextcloud:28.0
    restart: always
    container_name: Nextcloud
    volumes:
      - ./data/nextcloud:/var/www/html
    environment:
      - MYSQL_DATABASE Nextcloud
      - MYSQL_USER NcUser
      - MYSQL_PASSWORD NcUser
      - MYSQL_HOST nc-mariaddb
      - NEXTCLOUD_MEMORY_LIMIT 2048M
      - REDIS_HOST=redis
    networks:
      - wirdockenan.local
    ports:
      - "88:80"

  nc-mariadb:
    image: mariadb:11.2.2
    restart: always
    volumes:
      - ./data/mariadb-data:/var/lib/mariadb
      - ./data/mysql-data:/var/lib/mysql
    # Hatte damit versucht den Port zu ändern.
    # - type: bind
    #   source: ./data/mariadb-conf/mariadb.cnf
    #   target: /etc/mysql/mariadb.cnf
    environment:
      MARIADB_ROOT_PASSWORD: Admin
      MARIADB_DATABASE: Nextcloud
      MARIADB_USER: NcUser
      MARIADB_PASSWORD: NcUser
    # Dieser ist es. Damit hat er den Port geändert.
      MYSQL_TCP_PORT: 3366
    networks:
      - wirdockenan.local
    ports:
      - 3366:3306
    expose:
      - 3366
```

## Moodle

```docker
version: '3'
networks:
  wirdockenan.local:
    external: true
services:
  moodle:
    image: bitnami/moodle:4.3.2
    ports:
    # Benutze nur port 443  
    # - '82:8080'
      - '443:8443'
    environment:
    # Diesen Code nur hinzufügen, wenn eine Moodle Datenbank schon existiert.
    # - MOODLE_SKIP_BOOTSTRAP
      - MOODLE_USERNAME=Admin
      - MOODLE_PASSWORD=Admin
      - MOODLE_SITE_NAME=WirDockenAn Moodle
      - MOODLE_DATABASE_HOST=mo-mariadb
      - MOODLE_DATABASE_PORT_NUMBER=3306
      - MOODLE_DATABASE_NAME=moodle
      - MOODLE_DATABASE_USER=MoUser
      - MOODLE_DATABASE_PASSWORD=MoUser
      - MOODLE_LANG=de
      - PHP_ENABLE_OPCACHE=true
      - PHP_MEMORY_LIMIT=512M
    volumes:
      - ./data/moodledata-data:/bitnami/moodledata
      - ./data/moodle_data:/bitnami/moodle
    # Falls man in der httpd.conf Code ändern möchte muss man ihm zuerst eine lokal bestehende datei mappen
    # - ./data/httpd/httpd.conf:/opt/bitnami/apache/conf/httpd.conf
    networks:
      - wirdockenan.local

  mo-mariadb:
    image: mariadb:11.2.2
    restart: always
    volumes:
      - ./data/mariadb-data:/var/lib/mariadb
      - ./data/mysql-data:/var/lib/mysql
    environment:
      MARIADB_ROOT_PASSWORD: Admin
      MARIADB_DATABASE: moodle
      MARIADB_USER: MoUser
      MARIADB_PASSWORD: MoUser
    networks:
      - wirdockenan.local
    ports:
      - 3306:3306
```

## Redis

```docker
version: '2'
networks:
  wirdockenan.local:
    external: true
services:
  redis:
    image: 'bitnami/redis:latest'
    environment:
      - ALLOW_EMPTY_PASSWORD=yes
    volumes:
      - ./data/Redis:/bitnami/redis/data
    networks:
      - wirdockenan.local
```

## Adminer

```docker
version: '3'
networks:
  wirdockenan.local:
    external: true
services:
  # Wird genutzt um beide MariaDB Datenbanken überschauen zu können.
  adminer:
    image: adminer
    restart: always
    networks:
      - wirdockenan.local
    ports:
      - 8008:8080
```