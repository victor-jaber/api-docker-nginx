# Configuração do Docker Swarm e Proxy Reverso com NGINX

## 📦 Iniciar API e MongoDB com Proxy Reverso

Supondo que você tenha seguido os passos anteriores, importe os arquivos conforme estão no repositório. Lembrando que só há necessidade de adicioná-los.

### 🐳 Configuração do Docker Swarm

Fizemos algumas alterações no `docker-swarm.yml` e no `app.conf`. No `docker-swarm.yml`, adicionamos:

```yaml
api:
  build:
    context: ./py-api
    dockerfile: Dockerfile
    image: yourusernamedockerhub/sf-api
  environment:
    - FLASK_ENV=production # ou outro ambiente adequado
    - MONGO_INITDB_ROOT_USERNAME=your_username
    - MONGO_INITDB_ROOT_PASSWORD=mypass
    - MONGO_URI=mongodb://your_username@mongodb:27017/your_database
  networks:
    - net_swarm
  deploy:
    replicas: 1
    resources:
      limits:
        cpus: "1"
        memory: 1G
      reservations:
        cpus: "0.25"
        memory: 50M
    restart_policy:
      condition: on-failure
  depends_on:
    - mongodb

mongodb:
  image: mongo:4.4.6
  container_name: mongodb
  ports:
    - "27017:27017"
  environment:
    MONGO_INITDB_ROOT_USERNAME: your_username
    MONGO_INITDB_ROOT_PASSWORD: mypass
  volumes:
    - mongo_data:/data/db
  networks:
    - net_swarm

networks:
  net_swarm:
    external: true
    name: net_swarm

volumes:
  mongo_data:
```

# 🌐 Configuração do Proxy Reverso no NGINX

## 📝 Arquivo `app.conf`:

```nginx
server {
    server_name nginx.devsipremo.com;

    root /usr/share/nginx/html;
    index index.html index.htm;

    location / {
        try_files $uri $uri/ /index.html;
        proxy_pass http://react-app:80;  # Direciona para o serviço react-app
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location /api/ {
        proxy_pass http://api:8000/;  # Ajuste a porta se necessário
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    location ~* \.(css|js|jpg|jpeg|png)$ {
        expires 6h; # Exemplos: max, 30m, 1y, off
        log_not_found off;
    }

    listen 443 ssl; # managed by Certbot
    ssl_certificate /etc/letsencrypt/live/nginx.devsipremo.com/fullchain.pem; # managed by Certbot
    ssl_certificate_key /etc/letsencrypt/live/nginx.devsipremo.com/privkey.pem; # managed by Certbot
    include /etc/letsencrypt/options-ssl-nginx.conf; # managed by Certbot
    ssl_dhparam /etc/letsencrypt/ssl-dhparams.pem; # managed by Certbot
}

server {
    if ($host = nginx.devsipremo.com) {
        return 301 https://$host$request_uri;
    } # managed by Certbot

    listen 80;
    server_name nginx.devsipremo.com;

    # Redireciona todas as requisições HTTP para HTTPS
    location / {
        return 301 https://$host$request_uri;
    }
}
```
## 🚀 Executando a Configuração

Feito isso, é só rodar o comando `sh sh-up.sh` novamente e pronto! Tudo estará funcionando.
