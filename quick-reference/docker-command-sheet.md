# Docker Command Sheet — Ambiente de Desenvolvimento

Um guia rápido com comandos Docker usados no dia a dia de desenvolvimento.  
Todos os exemplos usam placeholders como `<image_name>`, `<container_name>`, `<path_to_dockerfile>`, `<host_port>`, `<container_port>`, etc., para você adaptar ao seu contexto.

---

## Sumário

- [Comandos Básicos](#comandos-básicos)  
- [Volumes](#volumes)  
- [Redes (Networking)](#redes-networking)  
- [Trabalhando com Docker Hub ou Registro de Imagens](#trabalhando-com-docker-hub-ou-registro-de-imagens)  
- [Dicas e Boas Práticas](#dicas-e-boas-práticas)

---

## Comandos Básicos

| Ação | Comando |
|---|---|
| Ver versão instalada do Docker | `docker version` |
| Puxar imagem do Docker Hub | `docker pull <image_name>` |
| Construir imagem a partir de um Dockerfile | `docker build -t <image_name> <path_to_dockerfile>` |
| Rodar container interativamente (modo interativo) | `docker run -it <image_name>` |
| Rodar container mapeando portas (host → container) | `docker run -p <host_port>:<container_port> <image_name>` |
| Dar um nome customizado para o container | `docker run -it --name <container_name> <image_name>` |
| Listar containers em execução | `docker container ls` |
| Listar todos os containers (inclusive parados) | `docker container ls -a` |
| Parar um container | `docker stop <container_id_or_name>` |
| Remover um container | `docker container rm <container_id_or_name>` |
| Listar imagens disponíveis localmente | `docker image ls` |
| Remover uma imagem | `docker image rm <image_id_or_name>` |

---

## Volumes

| Ação | Comando |
|---|---|
| Criar volume nomeado | `docker volume create <volume_name>` |
| Listar volumes | `docker volume ls` |
| Rodar container montando diretório local | `docker run -it -v <host_path>:<container_path> <image_name>` |
| Rodar container usando volume nomeado | `docker run -it -v <volume_name>:<container_path> <image_name> bash` |

---

## Redes (Networking)

| Ação | Comando |
|---|---|
| Listar redes existentes | `docker network ls` |
| Inspecionar rede padrão (bridge) | `docker inspect bridge` |
| Criar nova rede | `docker network create <network_name>` |
| Rodar container em rede específica | `docker run -it --network <network_name> --name <container_name> <image_name>` |
| Conectar container já existente a rede | `docker network connect <network_name> <container_name>` |
| Executar comando dentro de container em execução | `docker exec -it <container_name> bash` |

---

## Trabalhando com Docker Hub ou Outro Registro de Imagens

| Ação | Comando |
|---|---|
| Construir imagem com tag para Docker Hub ou outro registry | `docker build -t <registry>/<username>/<repo_name>:<tag> <path_to_dockerfile>` |
| Push da imagem para o registro | `docker push <registry>/<username>/<repo_name>:<tag>` |
| Pull da imagem do registro | `docker pull <registry>/<username>/<repo_name>:<tag>` |
| Rodar container a partir da imagem do registro | `docker run -it <registry>/<username>/<repo_name>:<tag>` |

---

## Dicas e Boas Práticas

- Sempre adicionar tags em imagens para facilitar versionamento (`:latest`, `:v1.0`, etc.).  
- Evitar usar `latest` em produção; usar tags específicas.  
- Limpar imagens e containers que não forem mais usados para liberar espaço:  
  ```bash
  docker container prune
  docker image prune
  docker volume prune
  docker network prune
  ```  
- Usar `docker-compose` para orquestrar múltiplos containers/localizações de desenvolvimento, se aplicável.  
- Versionar Dockerfiles e arquivos de configuração (ex: `.dockerignore`).  
- Revisar tamanho das imagens; priorizar imagens menores (ex: usar imagens base `alpine` quando possível).

---

> _Referência adaptada de “Docker Command Sheet — My Go‑To Dev Environment” por Devashish Roy._ ([dev.to](https://dev.to/roydevashish/docker-command-sheet-my-go-to-dev-environment-2kpg))
