# Padrão Blue/Green Deployment

Estratégia de implantação que mantém duas versões idênticas do ambiente (Blue e Green), alternando entre elas para atualizações sem interrupção.

## Sumário

- [Padrão Blue/Green Deployment](#padrão-blue-green-deployment)
  - [Componentes](#componentes)
  - [Benefícios](#benefícios)
  - [Contras](#contras)
  - [Quando Usar](#quando-usar)
  - [Exemplo com Docker e Nginx](#exemplo-com-docker-e-nginx)

## Componentes

- **Ambiente Blue**: Versão atual em produção, atendendo tráfego.
- **Ambiente Green**: Nova versão, testada e pronta para implantação.
- **Roteador/Balanceador**: Direciona tráfego entre Blue e Green (ex.: Nginx, Kubernetes Ingress).
- **Orquestrador**: Gerencia a troca e implantação (ex.: Kubernetes, Docker Compose).

## Benefícios

- Zero downtime: Tráfego é alternado instantaneamente.
- Rollback fácil: Reverte para Blue em caso de falhas.
- Testes seguros: Green é testado antes de receber tráfego.
- Consistência: Ambientes idênticos reduzem riscos.

## Contras

- Custo: Requer duplicação de recursos (servidores, contêineres).
- Complexidade: Configuração e sincronização entre ambientes.
- Banco de dados: Sincronização de schema pode ser desafiadora.
- Consumo de recursos: Manter dois ambientes ativos.

## Quando Usar

Use para aplicações críticas que exigem alta disponibilidade, atualizações frequentes e rollback rápido (ex.: e-commerce, APIs públicas).

## Exemplo com Docker e Nginx

### Dockerfile do Serviço

```dockerfile
FROM openjdk:17-jdk-slim
WORKDIR /app
COPY target/app.jar app.jar
EXPOSE 8080
ENTRYPOINT ["java", "-jar", "app.jar"]
```

### Docker Compose (Blue e Green)

```yaml
version: '3.8'
services:
  blue:
    image: myapp:latest
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8081:8080"
    environment:
      - APP_VERSION=blue
  green:
    image: myapp:latest
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "8082:8080"
    environment:
      - APP_VERSION=green
  nginx:
    image: nginx:latest
    volumes:
      - ./nginx.conf:/etc/nginx/nginx.conf
    ports:
      - "80:80"
    depends_on:
      - blue
      - green
```

### Configuração do Nginx (Roteador)

```nginx
events {}
http {
    upstream blue {
        server blue:8080;
    }
    upstream green {
        server green:8080;
    }
    server {
        listen 80;
        location / {
            proxy_pass http://blue; # Alternar para green na implantação
            proxy_set_header Host $host;
            proxy_set_header X-Real-IP $remote_addr;
        }
    }
}
```