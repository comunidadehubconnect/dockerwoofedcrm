<p align="center">
<img src="https://cwmkt.com.br/wp-content/uploads/2024/04/logo_github.png" width="240" />
<p align="center">Seja bem-vindo ao Guia de Instalação Docker Quepasa 🚀</p>
</p>
  
<p align="center"> 
<a href="https://hubconnect.top" target="_blank">👉 Participe da Comunidade HubConnect 👈</a>
</p>

<hr />
<hr />

### Caso não tenha Portainer e Traefix instalado, siga primeira etapa

<details>
<summary>Instalando Portainer e Traefix</summary>

### Atualizando Dependências

Atualize os repositórios do Ubuntu executando o seguinte comando:

```bash
sudo apt update && apt upgrade -y
```

----------------------------------------------------------------------------

**Instale o Docker em sua VPS**

```bash
sudo apt install docker.io -y
```

----------------------------------------------------------------------------

**Instalando Portainer**

```bash
docker swarm init
```

```bash
nano traefik.yml
```

```bash
version: "3.8"

services:

  traefik:
    image: traefik:v2.11.0
    command:
      - "--api.dashboard=true"
      - "--providers.docker.swarmMode=true"
      - "--providers.docker.endpoint=unix:///var/run/docker.sock"
      - "--providers.docker.exposedbydefault=false"
      - "--providers.docker.network=ecosystem_network"
      - "--entrypoints.web.address=:80"
      - "--entrypoints.web.http.redirections.entryPoint.to=websecure"
      - "--entrypoints.web.http.redirections.entryPoint.scheme=https"
      - "--entrypoints.web.http.redirections.entrypoint.permanent=true"
      - "--entrypoints.websecure.address=:443"
      - "--certificatesresolvers.letsencryptresolver.acme.httpchallenge=true"
      - "--certificatesresolvers.letsencryptresolver.acme.httpchallenge.entrypoint=web"
      - "--certificatesresolvers.letsencryptresolver.acme.email=contato@seudominio.com.br"
      - "--certificatesresolvers.letsencryptresolver.acme.storage=/etc/traefik/letsencrypt/acme.json"
      - "--log.level=DEBUG"
      - "--log.format=common"
      - "--log.filePath=/var/log/traefik/traefik.log"
      - "--accesslog=true"
      - "--accesslog.filepath=/var/log/traefik/access-log"
    deploy:
      placement:
        constraints:
          - node.role == manager
      labels:
        - "traefik.enable=true"
        - "traefik.http.middlewares.redirect-https.redirectscheme.scheme=https"
        - "traefik.http.middlewares.redirect-https.redirectscheme.permanent=true"
        - "traefik.http.routers.http-catchall.rule=hostregexp(`{host:.+}`)"
        - "traefik.http.routers.http-catchall.entrypoints=web"
        - "traefik.http.routers.http-catchall.middlewares=redirect-https@docker"
        - "traefik.http.routers.http-catchall.priority=1"
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock:ro"
      - "traefik_certificates_volume:/etc/traefik/letsencrypt"
    ports:
      - target: 80
        published: 80
        mode: host
      - target: 443
        published: 443
        mode: host
    networks:
      - ecosystem_network

volumes:
  traefik_certificates_volume:
    external: true
    name: traefik_certificates_volume

networks:
  ecosystem_network:
    external: true
    name: ecosystem_network
 ```

```bash
nano portainer.yml
```

```bash
version: "3.8"

services:

  agent:
    image: portainer/agent:latest
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock
      - /var/lib/docker/volumes:/var/lib/docker/volumes
    networks:
      - ecosystem_network
    deploy:
      mode: global
      placement:
        constraints: [node.platform.os == linux]

  portainer:
    image: portainer/portainer-ce:latest
    command: -H tcp://tasks.agent:9001 --tlsskipverify
    volumes:
      - portainer_volume:/data
    networks:
      - ecosystem_network
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints: [node.role == manager]
      labels:
        - "traefik.enable=true"
        - "traefik.docker.network=ecosystem_network"
        - "traefik.http.routers.portainer.rule=Host(`painel.seudominio.com.br`)"
        - "traefik.http.routers.portainer.entrypoints=websecure"
        - "traefik.http.routers.portainer.priority=1"
        - "traefik.http.routers.portainer.tls.certresolver=letsencryptresolver"
        - "traefik.http.routers.portainer.service=portainer"
        - "traefik.http.services.portainer.loadbalancer.server.port=9000"

networks:
  ecosystem_network:
    external: true
    attachable: true
    name: ecosystem_network

volumes:
  portainer_volume:
    external: true
    name: portainer_volume
```

```bash
docker network create --driver=overlay ecosystem_network
```

```bash
docker stack deploy --prune --resolve-image always -c traefik.yml traefik
```

```bash
docker stack deploy --prune --resolve-image always -c portainer.yml portainer
```

Acesse URL de seu Site e Crie Usuario


</details>


Postgres

### Adicione Stack abaixo, Stack > add stack

![image](https://github.com/cwmkt/dockerquepasa/assets/91642837/623a6dc6-c231-4105-9a02-3070d894adb8)

```bash
version: "3.8"

services:

  postgresql:
    image: bitnami/postgresql:15
    restart: always
    environment:
      POSTGRESQL_USERNAME: postgres
      POSTGRESQL_PASSWORD: r45796yv3bhub9w4f3ga3ikxmxos648r
    networks:
      - ecosystem_network
    ports:
      - 5432:5432
    volumes:
      - postgresql_volume:/bitnami/postgresql
      - postgresql_config_volume:/opt/bitnami/postgresql/conf
    deploy:
    

volumes:
  postgresql_volume:
    external: true
    name: postgresql_volume
  postgresql_config_volume:
    external: true
    name: postgresql_config_volume
    
networks:
  ecosystem_network:
    external: true
    name: ecosystem_network
```

Depois clique em DEPLOY

![image](https://github.com/cwmkt/dockerquepasa/assets/91642837/bdc62781-993a-4d31-b8cd-5cd6466900f5)

Postgres
Acesse portainer, vá até contaneir Postgres, crie banco de dados

![image](https://github.com/cwmkt/woofedcrm/assets/91642837/503bf33f-ff42-4fe5-9a8f-a98e2d80d6e4)

![image](https://github.com/cwmkt/woofedcrm/assets/91642837/67eb98c2-f7e7-4f27-ae9d-1befc672edcf)


```bash
psql -U postgres
```

```bash
create database woofedcrm;
```

![image](https://github.com/cwmkt/woofedcrm/assets/91642837/09cf38c6-7c59-4a2f-a2f2-e37341d9d15e)

Redis
Para instalar o redis vamos fazer as mesmas etapas que fizemos no postgres: Vamos para aba Stacks e depois em add Stack

![image](https://github.com/cwmkt/woofedcrm/assets/91642837/07c67cb5-465c-4e05-88e7-164ec8456f00)

Colocamos o nome da stack (recomendo deixar reids)

![image](https://github.com/cwmkt/woofedcrm/assets/91642837/03d15854-59bc-47df-a983-ad9adf0c93c4)

Cole a stack do redis abaixo no seu portainer:

```bash
version: '3.7'

services:
  redis:
    image: redis:latest
    command: [
        "redis-server",
        "--appendonly",
        "yes",
        "--port",
        "6379"
      ]
    volumes:
      - redis_data:/data
    networks:
      - SuaRede ## ---> NOME DA REDE INTERNA <--- ##
    deploy:
      placement:
        constraints:
          - node.role == manager

volumes:
  redis_data:
    external: true
    name: redis_woofedcrm_data

networks:
  SuaRede: ## ---> NOME DA REDE INTERNA <--- ##
    external: true
    name: SuaRede ## ---> NOME DA REDE INTERNA <--- ##
```

![image](https://github.com/cwmkt/woofedcrm/assets/91642837/dedb5386-bc42-465c-a39e-1ff2aa131f85)

e pronto, o redis esta instalado e funcionando na sua VPS

WoofedCRM

Para instalar o WoofedCRM vamos fazer as mesmas etapas que fizemos no postgres e no Redis. Vamos para aba Stacks e depois em add Stack

![image](https://github.com/cwmkt/woofedcrm/assets/91642837/d1e54ab7-5c5f-4c28-902f-27266ed0abb7)

Colocamos o nome da stack (recomendo deixar woofedcrm)

![image](https://github.com/cwmkt/woofedcrm/assets/91642837/84329d5f-1780-4b6f-b91a-5a2eed6a2bc8)

Cole a stack do WoofedCRM abaixo no seu portainer:
Encontre a versão do woofedcrm aqui: https://hub.docker.com/r/douglara/woofedcrm/tags

```bash
version: '3.7'

services:
  woofedcrm:
    image: douglara/woofedcrm:development-4f2ba85549a8e897159f17d87d67f86693166ebb
    command: bundle exec rails s -p 3000 -b 0.0.0.0
    networks:
      - sua_rede
    volumes:
      - woofedcrm_data:/app
    environment:
      - ENABLE_USER_SIGNUP=true
      - RAILS_ENV=production
      - RACK_ENV=production
      - NODE_ENV=production
      - MOTOR_AUTH_USERNAME=Usuario
      - MOTOR_AUTH_PASSWORD=SUASENHA
      - FRONTEND_URL=https://Seudominio.com.br
      - DATABASE_URL=postgresql://postgres:SENHAPOSTGRES@postgresql:5432/BANCOPOSTGRES
      - REDIS_URL=redis://redis:6379/0
      - ACTIVE_STORAGE_SERVICE=local
      - RAILS_LOG_LEVEL=debug
      - DEFAULT_TIMEZONE=Brasilia

    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints:
          - node.role == manager

      labels:
        - traefik.enable=true
        - traefik.http.routers.woofedcrm.rule=Host(`Seudominio.com.br`)
        - traefik.http.routers.woofedcrm.entrypoints=websecure
        - traefik.http.routers.woofedcrm.tls.certresolver=letsencryptresolver
        - traefik.http.routers.woofedcrm.priority=1
        - traefik.http.routers.woofedcrm.service=woofedcrm
        - traefik.http.services.woofedcrm.loadbalancer.server.port=3000
        - traefik.http.services.woofedcrm.loadbalancer.passhostheader=true
        - traefik.http.middlewares.sslheader.headers.customrequestheaders.X-Forwarded-Proto=https
        - traefik.http.routers.woofedcrm.middlewares=sslheader@docker

  sidekiq:
    image: douglara/woofedcrm:development-c85cdfdd71bc0736e84bb29051aab16978d8c687
    command: bundle exec sidekiq -C config/sidekiq.yml
    networks:
      - sua_rede
    volumes:
      - woofedcrm_data:/app
    environment:
      - ENABLE_USER_SIGNUP=true
      - RAILS_ENV=production
      - RACK_ENV=production
      - NODE_ENV=production
      - MOTOR_AUTH_USERNAME=Usuario
      - MOTOR_AUTH_PASSWORD=SUASENHA
      - FRONTEND_URL=https://Seudominio.com.br
      - DATABASE_URL=postgresql://postgres:SENHAPOSTGRES@postgresql:5432/BANCOPOSTGRES
      - REDIS_URL=redis://redis:6379/0
      - ACTIVE_STORAGE_SERVICE=local
      - RAILS_LOG_LEVEL=debug
      - DEFAULT_TIMEZONE=Brasilia
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints:
          - node.role == manager


  good_job:
    image: douglara/woofedcrm:development-c85cdfdd71bc0736e84bb29051aab16978d8c687
    command: bundle exec good_job
    networks:
      - sua_rede
    volumes:
      - woofedcrm_data:/app
    environment:
      - ENABLE_USER_SIGNUP=true
      - RAILS_ENV=production
      - RACK_ENV=production
      - NODE_ENV=production
      - MOTOR_AUTH_USERNAME=Usuario
      - MOTOR_AUTH_PASSWORD=SUASENHA
      - FRONTEND_URL=https://Seudominio.com.br
      - DATABASE_URL=postgresql://postgres:SENHAPOSTGRES@postgresql:5432/BANCOPOSTGRES
      - REDIS_URL=redis://redis:6379/0
      - ACTIVE_STORAGE_SERVICE=local
      - RAILS_LOG_LEVEL=debug
      - DEFAULT_TIMEZONE=Brasilia
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints:
          - node.role == manager

volumes:
  woofedcrm_data:
    external: true
    name: woofedcrm_data

networks:
  sua_rede:
    external: true
    name: sua_rede
```

Obs: Lembre-se de alterar todos os campos comentados com as informações necessárias.
Após toda alteração, podemos desativar o Enable access control e clicar em Deploy the stack

![image](https://github.com/cwmkt/woofedcrm/assets/91642837/e39d7915-e223-4afe-98d0-fdc369138265)

Após isso vamos para a seção de containers

![image](https://github.com/cwmkt/woofedcrm/assets/91642837/40d35df4-1a69-47e5-98d5-f2194d5cd580)

Precisamos clicar a opção de console do container woofedcrm_woofedcrm

![image](https://github.com/cwmkt/woofedcrm/assets/91642837/5bc50b8d-85fe-450b-ab50-b7d4c43003ac)

Nessa nova tela podemos clicar em Connect

![image](https://github.com/cwmkt/woofedcrm/assets/91642837/8dc4e377-45cb-45b5-bcd1-327b7e91ceed)

Por fim no terminal que abriu vamos colocar esses dois comandos:

```bash
rails db:create
```

```bash
rails db:migrate
```

Vai aparecer um monte de informações no terminal, mas é normal.
Pronto agora você já terá acesso ao WoofedCRM

Url Cliente

https://seudominio.com.br/users/sign_in

https://seudominio.com.br/motor_admin/

![image](https://github.com/cwmkt/woofedcrm/assets/91642837/d86b5c4f-13c5-4ff3-be9a-d9310a13ca71)

Lembre-se de criar uma conta para cada empresa.

**Pronto tudo Funcionando** ✅😎







