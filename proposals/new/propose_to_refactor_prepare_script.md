# Proposal: Refactor Prepare Script



Author: Qian Deng



##  Abstract

Refactor the prepare script code, make it eazy to configure and change the template rendering engine to jinja2 and containerize this prepare file.



## Overview

Currently, the prepare file is too giant and messy to understand the logic inside it. So need refacor this file make it more stylish and modular to decease the learning curve for developer. 

In addition, the template we just using is just a simple string subsitude provided by awk. My opinion is change it to *jinja2* which is widely used in python community. 

Futhermore, there was a historical issues that cannot change `/data` directory through config file. Therefore we need solve this issue in this refactor. 

Moreover, the configparse format is old-fashine and cannot present complex structure, also in this refactor  the `harbor.cfg` file will replaced by `harbor.yml` .

 Finally, all the codes and it dependencies will packaged in one container to realease.



## Proposal

1. Refactor code implementation make it more pythonic and structure, the code skeleton looks like this:

   1. ``` 
      ├── g
      │   ├── aaa.py
      │   ├── bbb.py
      │   └── __init__.py
      ├── libs
      │   ├── ccc.py
      │   ├── ddd.py
      │   └── __init__.py
      ├── main.py
      └── requirement.txt
      ```

2. The template engine is essential to render config files, a high level rendering engine can solve some strange syntax in current `*.tpl` files and can generated just one `docker-compose.yml` file to cover all situations. The templates may looks like this:

3. ```jinja2
   PORT=8080
   LOG_LEVEL=info
   EXT_ENDPOINT={{public_url}}
   AUTH_MODE={{auth_mode}}
   SELF_REGISTRATION={{self_registration}}
   LDAP_URL={{ldap_url}}
   LDAP_SEARCH_DN={{ldap_searchdn}}
   LDAP_SEARCH_PWD={{ldap_search_pwd}}
   LDAP_BASE_DN={{ldap_basedn}}
   LDAP_FILTER={{ldap_filter}}
   LDAP_UID={{ldap_uid}}
   LDAP_SCOPE={{ldap_scope}}
   LDAP_TIMEOUT={{ldap_timeout}}
   LDAP_VERIFY_CERT={{ldap_verify_cert}}
   DATABASE_TYPE=postgresql
   POSTGRESQL_HOST={{db_host}}
   POSTGRESQL_PORT={{db_port}}
   POSTGRESQL_USERNAME={{db_user}}
   ```

   and this:

   ```jinja2
   version: '2'
   services:
     log:
       image: goharbor/harbor-log:{{version}}
       container_name: harbor-log 
       restart: always
       dns_search: .
       volumes:
         - /var/log/harbor/:/var/log/docker/:z
         - ./common/config/log/:/etc/logrotate.d/:z
       ports:
         - 127.0.0.1:1514:10514
       networks:
         - harbor
     registry:
         ...
     registryctl:
         ...
     postgresql:
       ...
     adminserver:
       ...
     core:
       ...
     portal:
       ...
   
     jobservice:
       ...
     redis:
       ...
     proxy:
       ...
   
   {% if with_notary %}
       notary-server:
         ...
       notary-signer:
         ...
   {% endif %}
   networks:
     harbor:
       external: false
   {% if with_notary %}
     harbor-notary:
       external: false
     notary-sig:
       external: false
   {% endif %}
   
   ```

3. The `/data` dir is hardcoded in template file, this can also be sovled easily in replacing render engine.
4. Current config file is `harbor.cfg` which is configparser-style format. In order to describe more complex content, We need replace it using YAML format like `harbor.yml` .
5. Packaging all files and its denpendencies into a container.  All these codes and changes is related to python, as a result , we should using official python images as the base image in Dockerfile.