# stacks-docker-swarm

Aqui você vai encontrar todas minhas stacks com Traefik em ambiente Docker Swarm.

<details>
<summary>Stack: evolution_api</summary>

### Nome da stack:

```bash
evolution-api
```
### Conteúdo do arquivo docker swarm:

```bash
version: "3.7"

services:
  evolution_api:
    image: atendai/evolution-api:homolog
    command: ["node", "./dist/src/main.js"]
    networks:
      - rede_interna
    volumes:
    - evolution_api_instances:/evolution/instances
    - evolution_api_store:/evolution/store
    - evolution_api_views:/evolution/views
    environment:
      - SERVER_URL=https://seudominio.com.br
      - DOCKER_ENV=true
      - LOG_LEVEL=ERROR,WARN,DEBUG,INFO,LOG,VERBOSE,DARK,WEBHOOKS
      - LOG_COLOR=true
      # Se você nem quiser uma expiração, insira o valor false
      - DEL_INSTANCE=false
      # Nome que será exibido na conexão do smartphone
      - CONFIG_SESSION_PHONE_CLIENT=EvolutionAPI
      # Nome do navegador = Chrome | Firefox | Edge | Opera | Safari
      - CONFIG_SESSION_PHONE_NAME=Chrome
      # Definir limite de exibição do qrcode
      - QRCODE_LIMIT=30
      - QRCODE_COLOR=#198754
      - AUTHENTICATION_TYPE=apikey
      - AUTHENTICATION_API_KEY=crie_sua_apikey
      - AUTHENTICATION_EXPOSE_IN_FETCH_INSTANCES=true
      ## Defina a chave secreta para criptografar e descriptografar seu token e seu prazo de validade
      # segundos - 3600s ===1h | zero (0) - nunca expira
      - AUTHENTICATION_JWT_EXPIRIN_IN=0
      - AUTHENTICATION_JWT_SECRET='crie_sua_secret'
      # Armazenamento temporário de dados
      - STORE_MESSAGES=true
      - STORE_MESSAGE_UP=true
      - STORE_CONTACTS=true
      - STORE_CHATS=true
      # Definir intervalo de armazenamento em segundos (7200 = 2h)
      - CLEAN_STORE_CLEANING_INTERVAL=7200 # seconds === 2h
      - CLEAN_STORE_MESSAGES=true
      - CLEAN_STORE_MESSAGE_UP=true
      - CLEAN_STORE_CONTACTS=true
      - CLEAN_STORE_CHATS=true
      ## # Armazenamento permanente de dados
      # Ativar Banco de Dados
      - DATABASE_ENABLED=true
      - DATABASE_CONNECTION_URI=mongodb://root:root@ip_da_vps:27017/?authSource=admin&readPreference=primary&ssl=false&directConnection=true
      - DATABASE_CONNECTION_DB_PREFIX_NAME=evolution
      # Escolha os dados que deseja salvar no banco de dados ou armazenamento do aplicativo
      - DATABASE_SAVE_DATA_INSTANCE=true
      - DATABASE_SAVE_DATA_NEW_MESSAGE=true
      - DATABASE_SAVE_MESSAGE_UPDATE=true
      - DATABASE_SAVE_DATA_CONTACTS=true
      - DATABASE_SAVE_DATA_CHATS=true
      # REDIS (na versão 1.6.0 ainda em beta para multissessão, mas você pode ativar no modo de sessão única)
      - REDIS_ENABLED=false
      - REDIS_URI=redis://redis:6379
      - REDIS_PREFIX_KEY=evdocker
      # Configuraçãoes do RabbitMQ
      - RABBITMQ_ENABLED=true
      - RABBITMQ_URI=amqp://adm:sua_senha@rabbitmq:5672
      - WEBSOCKET_ENABLED=false
      #- SQS_ENABLED=false
      #- SQS_ACCESS_KEY_ID=
      #- SQS_SECRET_ACCESS_KEY=
      #- SQS_ACCOUNT_ID=
      #- SQS_REGION=
      # Configurações globais de webhook
      - WEBHOOK_GLOBAL_URL=https://URL
      - WEBHOOK_GLOBAL_ENABLED=false
      # Com esta opção ativada você trabalha com uma url por evento de webhook, respeitando a url global e o nome de cada evento
      - WEBHOOK_GLOBAL_WEBHOOK_BY_EVENTS=false
      # Defina os eventos que você deseja ouvir
      - WEBHOOK_EVENTS_APPLICATION_STARTUP=false
      - WEBHOOK_EVENTS_QRCODE_UPDATED=true
      # EVENTOS DE MESSAGENS
      - WEBHOOK_EVENTS_MESSAGES_SET=true
      - WEBHOOK_EVENTS_MESSAGES_UPSERT=true
      - WEBHOOK_EVENTS_MESSAGES_UPDATE=true
      - WEBHOOK_EVENTS_MESSAGES_DELETE=true
      - WEBHOOK_EVENTS_SEND_MESSAGE=true
      # EVENTOS DE CONTATOS
      - WEBHOOK_EVENTS_CONTACTS_SET=true
      - WEBHOOK_EVENTS_CONTACTS_UPSERT=true
      - WEBHOOK_EVENTS_CONTACTS_UPDATE=true
      # EVENTO DE PRESENÇA
      - WEBHOOK_EVENTS_PRESENCE_UPDATE=true
      # EVENTOS DE CONVERSAS
      - WEBHOOK_EVENTS_CHATS_SET=true
      - WEBHOOK_EVENTS_CHATS_UPSERT=true
      - WEBHOOK_EVENTS_CHATS_UPDATE=true
      - WEBHOOK_EVENTS_CHATS_DELETE=true
      # EVENTOS DE GRUPOS
      - WEBHOOK_EVENTS_GROUPS_UPSERT=true
      - WEBHOOK_EVENTS_GROUPS_UPDATE=true
      - WEBHOOK_EVENTS_GROUP_PARTICIPANTS_UPDATE=true
      # ENVENTO DE CONECÇÃO
      - WEBHOOK_EVENTS_CONNECTION_UPDATE=true
      # ENVENTO DE CHAMADA
      - WEBHOOK_EVENTS_CALL=true
      # Este evento é acionado sempre que um novo token é solicitado por meio da rota de atualização
      - WEBHOOK_EVENTS_NEW_JWT_TOKEN=false
      # Ajustes Typebot 2.20.0
      - TYPEBOT_API_VERSION=latest  # old | latest
      - TYPEBOT_KEEP_OPEN=true # mantem aberta a sessão do bot para não reiniciar
      - SERVER_TYPE=http
      - SERVER_PORT=8080
      - WEBHOOK_EVENTS_TYPEBOT_START=false
      - WEBHOOK_EVENTS_TYPEBOT_CHANGE_STATUS=false
      # Este evento é usado com Chama AI
      - WEBHOOK_EVENTS_CHAMA_AI_ACTION=false
      # Este evento é usado para enviar erros
      - WEBHOOK_EVENTS_ERRORS=false
      - WEBHOOK_EVENTS_ERRORS_WEBHOOK=
    deploy:
      mode: replicated
      replicas: 1
      placement:
        constraints:
          - node.role == manager
      resources:
        limits:
          cpus: "1"
          memory: 1024M
      labels:
        - traefik.enable=true
        - traefik.http.routers.evolution_api.rule=Host(`seudominio.com.br`)
        - traefik.http.routers.evolution_api.entrypoints=websecure
        - traefik.http.routers.evolution_api.tls.certresolver=letsencryptresolver
        - traefik.http.routers.evolution_api.priority=1
        - traefik.http.routers.evolution_api.service=evolution_api
        - traefik.http.services.evolution_api.loadbalancer.server.port=8080
        - traefik.http.services.evolution_api.loadbalancer.passHostHeader=true
        - traefik.http.routers.evolution_api.tls=true

volumes:
  evolution_api_instances:
    external: true
    name: evolution_api_instances
  evolution_api_store:
    external: true
    name: evolution_api_store
  evolution_api_views:
    external: true
    name: evolution_api_views

networks:
  rede_interna:
    name: rede_interna
    external: true
```

</details>