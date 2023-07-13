

![[idpimage-1.png]]

Step-1: Setup Postgres db 
--------------------------------------------
We are going to use docker for deploy postgre db.
Create docker compose file & open compose file
```
touch docker-compose.yml
vim dockercompose.yml
```
Add below configuration in docker compose file
```
version: '3.3'
services:
  db:
    image: postgres:14.1-alpine
    restart: always
    environment:
      - POSTGRES_USER=postgres
      - POSTGRES_PASSWORD=postgres
    ports:
      - '5433:5432'
    volumes: 
      - /data/postgres-db-for-smartmf/data:/var/lib/postgresql/data
```

Now run docker compose file 
```
docker compose -f docker-compose.yml up -d
```

Postgres db is up now.

Check PostgreSQL db from docker service
`docker ps | grep postgres`

Output:
![[postgres.png]]


Step-2: Deploy IDP service
------------------------------------------------

Go to SSO directory & create script file for run idp container using docker.

```
#tocuh run.sh
#vim run.sh
```

add below configuration in run.sh  script file for up the docker service.
```
#!/usr/bin/env bash
IMAGE=docker.bracits.com/sbicloud/erp-idp:21.0.1

PORT=8080
CONTAINER=sso

docker pull $IMAGE
docker rm -f $CONTAINER

docker run -d --restart=always --network=host \
--name $CONTAINER \
-m 1G \
--env-file ./.env \
$IMAGE --optimized
```

Now create env file from idp directory and provided credentials as per server configuration.
For example, we have used OTC server sso .env file.

```
#touch .env
#vim .env
```

```
KC_DB_URL=jdbc:postgresql://10.150.150.2:5432/erp_sso_53_28
KC_DB_USERNAME=sso
KC_DB=postgres
KC_DB_PASSWORD=#asdasd34RT#
KC_PROXY=edge
KC_ENABLE_HTTP=true
KC_HOSTNAME_STRICT=false
DEBUG_MODE=true
DEBUG_PORT="*:5005"
QUARKUS_TRANSACTION_MANAGER_ENABLE_RECOVERY=true
KC_HTTP_PORT=8080
```

- Note: Here, we use postgres db credentials, make sure mentiond db user password and mentiond db  is created in Postgres DB.


Now run script for build &  deploy idp service.
```
./run.sh
```

IDP service is up now using  docker.
Check idp docker service
```
docker ps | grep idp
```

Step-3: Deploy Federation Service
---------------------------------------------------------
Create new directory for fedaration service
ex: `mkdir fedaration`

Go to federation directory create script file for deploy federation
```
touch run.sh
vim run.sh
```

Now, add below configuration on `run.sh` script.
```
#!/usr/bin/env bash

IMAGE=docker.bracits.com/sbicloud/user-federation:native
#IMAGE=docker.bracits.com/sbicloud/user-federation:latest

PORT=8078
CONTAINER=federation

docker pull $IMAGE
docker stop $CONTAINER
docker rm $CONTAINER

docker run -d --restart=always \
--name $CONTAINER \
-m 1G \
--network host \
-e PORT=$PORT \
-e DB_HOST=10.150.150.2 \
-e DB_PORT=4550 \
-e DB_NAME=sbicloud_bd \
-e DB_USER=bits \
-e DB_PASSWORD=biTS@\#123 \
-e API_USER=app \
-e API_PASSWORD=apass \
$IMAGE
```

Here, we use docker container for federation service and use  sbicloud_bd db.

Run mentioned script.
```
./run.sh
```

Federation deployment done. For check use below command.
```
docker ps | grep federation
```


Step-4: Config tomcat groovy file for enable idp service
---------------------------------------------------------------------------------------------
Now, open HRMS module groovy file & add below configuration for enable idp servcie.

For example, Here we use erp otc configuration, we have to configure as per project credential basis.

```
grails.plugin.springsecurity.providerNames = ['oidcAuthenticationProvider', 'daoAuthenticationProvider', 'anonymousAuthenticationProvider']                                         
sso.enable=true                                                                                                                                                   sso.serverUrl='https://erpotc.bracits.com/idp'                                                                                                              sso.realm='brac'                                                                                                                                                  sso.clientId='erp'                                                                                                                                                sso.adminClient='erp-admin'                                                                                                                                       sso.clientSecret='sT8dG4g2bvtR3TswRACn7sjnX8mzzZ2r'                                                                                                               sso.idp='sso'
```

Now Restart tomcat, IDP service is enabled.

idp admin login console url: `$https://sub-domain/idp/admin/

For example we use here otc idp.
`https://erpotc.bracits.com/idp/admin/`

IDP admin Console page:
![[Pasted image 20230713165028.png]]

Login Admin console & use realm: `brac` as we have 
This is idp admin console pag view, here we realm use brac as in our confiugration file we use `brac` realm. From cinfigure section we can configure Realm settings, authentication, identity provider & User federation. 
![[Pasted image 20230713165116.png]]

